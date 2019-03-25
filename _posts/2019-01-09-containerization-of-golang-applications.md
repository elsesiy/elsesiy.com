---
layout: post
title:  "Containerization of Golang applications"
categories: []
tags:
- docker
- container
- golang
- go
status: publish
type: post
published: true
meta: {}
---
I've been a working a lot in [Golang][golang] recently and even though it easily allows for single static binary compilation I find myself using [Docker][docker] a lot. Why? Well, especially when it comes to container orchestration and scaling of workloads some sort of container technology is used as these build the foundation of a Pod in [Kubernetes][k8s] (our deployment infrastructure).

While coming up with an initial Dockerfile is easy, correctly compiling a Go application and adhere to all sorts of container best practices is still hard. Does the size of the binary matter? If so, you might want to provide some additional flags during compilation. 
Do you build the app outside of the container and just copy the final artifact into it or do you use multi-stage builds to have everything within an isolated environment? How do you deal with caching of layers? How do you make sure that the final container is secure and only contains whatever is needed to execute your application? 
<!--more-->
The first thing I was overwhelmed with is how much old information is still out there and that even well-known developers such as [Tim Hockin][thockin-web] (co-creator of Kubernetes) are having [questions][thockin-twitter] on how to actually compile a Golang application *correctly*. As it turns out, some flags are [strictly unnecessary][gp-installsuffix] as of Go 1.10 but are still widely used. Ultimately, it all depends on your needs and whether you need [cgo][cgo] or not but even after studying a lot of blog posts I'm still not 100% sure about my approach. As it turns out, Tim created a nice [skeleton project][gh-go-build-template] which is a good starting point in my opinion.

Furthermore, I saw a lot of different approaches in terms of runtime base image and how the build process takes place. Some are using for `golang:alpine` with manually installing ca-certs, tzinfo, etc. during the build stage whereas others use plain `golang` instead. For the final stage common choices are either `scratch` or `alpine` which still provide a larger attack surface than i.e. `gcr.io/distroless/base`. As with many things, there's not a single *correct* approach because one might want to keep the ability to `docker exec -it` into a container around whereas others have better ways to debug their services.

While coming up with my current solution I had the following considerations to take into account. Local development should still be fast, the build process must be CI-friendly with clean & reproduceable builds and no additional tooling needed to secure the final image such as [microscanner][microscanner] or [clair][clair]. Hence, I created a `Makefile` that helps me take care of the heavy lifting and allows for fast local development where no Docker is used at all. A shortened & simplified version looks as follows:

```make
OUT := binary-name
PKG := github.com/package
VERSION := $(shell git describe --always --dirty)
PKG_LIST := $(shell go list ${PKG}/...)
GO_FILES := $(shell find . -name '*.go')

build:
	go build -i -v -o ${OUT} -ldflags="-X main.version=${VERSION}" ${PKG}

test:
	@go test -short ${PKG_LIST}

vet:
	@go vet ${PKG_LIST}

errorcheck:
	@errcheck ${PKG_LIST}

lint:
	@for file in ${GO_FILES} ;  do \
		golint $$file ; \
	done

container:
	@docker build --build-arg VERSION=${VERSION} -t registry/image:${VERSION} .

container-push:
	@docker push registry/image:${VERSION}

run: build
	./${OUT}

clean:
	-@rm ${OUT} ${OUT}-*

.PHONY: run build vet lint errorcheck
```

I'll talk about `-ldflags` in a bit, so don't worry about it for now. Since the regular `go build` command doesn't do static analysis on the project files, I created steps like `vet` (checks for correctness/suspicious constructs), `lint` (style mistakes) and `errorcheck` (missing error handling) I can run whenever I feel like it. This is not done implicitly through another step such as `build` because my CI system takes care of these things too. The rest of the file should be self-explanatory if you're familiar with make.  
Now, the following `Dockerfile` is only used in my CI system for which I don't mind it to fetch the dependencies during each build.

```docker
# Build stage
FROM golang:1.11.4 AS build-env

LABEL maintainer="Jonas-Taha El Sesiy <github@elsesiy.com>"

WORKDIR /project
ARG VERSION
COPY main.go go.mod go.sum ./
RUN bash -c "go get -d &> /dev/null" && \
    CGO_ENABLED=0 GOOS=linux go build -ldflags "-X main.version=${VERSION} -s -w" -a -o app .

# Final stage
FROM gcr.io/distroless/base
COPY --from=build-env /project/app .
CMD ["./app"]
```

I'm using multi-stage builds with the latest Golang version as the base image. For the final stage, I opted for [distroless][why-distroless] even though the final image is **bigger** than the other choices. Note that I'm using go modules for dependency management introduced in Go 1.11 for which I copy the `go.mod` and `go.sum` files into the container.  
As mentioned before, there are a couple of flags passed onto the go compiler via `-ldflags`. `-X main.version=abc` allows me to pass on the version information to the binary which is then used within the app in some fashion. `-s -w` disables the symbol table and the generation of debug data in form of DWARF in order to reduce the size of the binary which is useful for my production image.

This is just my take on this. If you have suggestions for improvements or any other remarks, please reach out. Thanks! :wave:

[golang]: https://golang.org/
[docker]: https://www.docker.com/
[k8s]: https://kubernetes.io/
[thockin-web]: http://www.hockin.org/~thockin/
[thockin-twitter]: https://twitter.com/thockin/status/758814480931229697?lang=en
[cgo]: https://golang.org/cmd/cgo/
[gh-go-build-template]: https://github.com/thockin/go-build-template
[gp-installsuffix]: https://plus.google.com/117192131596509381660/posts/eNnNePihYnK
[microscanner]: https://github.com/aquasecurity/microscanner
[clair]: https://github.com/coreos/clair
[why-distroless]: https://github.com/GoogleContainerTools/distroless#why-should-i-use-distroless-images
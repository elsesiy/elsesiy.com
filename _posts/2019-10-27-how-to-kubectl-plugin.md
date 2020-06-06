---
layout: post
title:  "Kubernetes: How to write a kubectl plugin"
categories: []
tags:
- krew
- kubectl
- plugin
- kubernetes
- k8s
- cncf
- hacktoberfest
status: publish
type: post
published: true
meta: {}
---
Hacktoberfest is almost over but since there're plenty of opportunities to contribute, I decided to take over the task of re-writing a `kubectl` plugin called `view-secret`.
For those of you who are not familar with `kubectl`, it's the CLI tool to work with [Kubernetes][kubernetes].
In this post I'd like to shed some light on [krew][krew] and what's necessary to create your very own plugin. Okay, so what's krew?

Krew is one of the many Kubernetes Special Interest Groups (SIG) and aims at solving the package management issue for `kubectl`. There's a limited amount of core functionality that ships with `kubectl` so krew is all about allowing developers to create their own extensions and contribute them back to the community. All available plugins are stored in the [krew-index][krew-index], a central repo that plugin maintainers use to publish/update their plugins. If you haven't used krew before, make sure to [install][install] it first.

<!--more-->
Some early plugins have been written in Bash and the maintainer was asking the community to take over the re-write in Go/Python and ultimately the maintenance of the plugins.
Since my day job requires me to create & manage Kubernetes-based deployments, I find myself in need of decoding secrets all the time.
A `secret` - _as the name may reveal already_ - is used to store secret information [Base64][Base64] encoded. This resource type is often used to populate environment configuration for deployments, to store docker registry auth information or tls secrets.

The typical workflow to decode a secret without `view-secret` looks as follows:

  1. `kubectl get secret <secret> -o yaml`
  2. Copy base64 encoded secret
  3. `echo "b64string" | base64 -d`

This gets quite cumbersome especially if you just want to check the entirety of a secret to see if everything looks ok.
There are solutions like [kubedecode][kubedecode] or the previous [view-secret][view-secret-old] implementation that aim at solving this problem but lack either native `kubectl` integration, are outdated/not maintained anymore or require you to always provide e.g. the `namespace` as a parameter.

So I went ahead and created a new implementation for [view-secret][view-secret] that is backward-compatible to the existing implementation but also adds a new capability, namely decoding all contents of a secret. My contribution has been [accepted][krew-contrib] and the plugin is available now, so let me walk you through the process.

As it turns out, creating your own plugin is super simple and well documented [here][krew-doc]. All you have to do is create a binary with the prefix `kubectl-`, make it executable and place it somewhere in your `$PATH`. A sample plugin can be as easy as this:

```bash
# kubectl-hello plugin
cat <<EOF > kubectl-hello
#!/usr/bin/env bash
echo "hello from krew"
EOF

# Make executable
chmod +x kubectl-hello

# Copy into some $PATH location
cp kubectl-hello /usr/local/bin

# Run plugin
kubectl hello
## prints "hello from krew"
```

Since my language of choice for this project was Go, I created a new project and added integrations such as [GoReleaser][go-releaser] to simplify shipping the binary for mulitple platforms and [Travis CI][travis-ci] to automate running builds/creating releases. To simplify the build/test process I also added a `Makefile`.
At this point my project repo had the following layout:

```bash
cmd
  kubectl-view-secret.go
pkg
  cmd
    view-secret.go
    view-secret_test.go
go.mod
go.sum
.goreleaser.yml
.travis.yml
Makefile
```

The established workflow was pretty straight-forward:

  1. push changes to master --> triggers travis to run tests
  2. tag commit --> triggers travis to use goreleaser

I previously [wrote][previous] about the usage of `Makefile`s in Go projects but for this project the targets are much simpler:

```bash
SOURCES := $(shell find . -name '*.go')
BINARY := kubectl-view-secret

build: kubectl-view-secret

test: $(SOURCES)
  go test -v -short -race -timeout 30s ./...

$(BINARY): $(SOURCES)
  CGO_ENABLED=0 go build -o $(BINARY) -ldflags="-s -w" ./cmd/$(BINARY).go
```

For the actual implementation I used [spf13/cobra][cobra] to parse flags and process user input. To get the secret contents I use `exec.Command` thus shelling out to the OS instead of using the kubernetes [go client][k8s-client-go] or [cli runtime][k8s-cli-runtime] as they add a huge overhead for such a small functionality.

After I finished the implementation, all I had to do was update the `plugins/view-secret.yaml` spec in the krew index to [use my new plugin][krew-contrib]. This meant changing the plugin description, the download links for the new binaries and the `sha256` checksums. Once the Pull Request got merged, the local plugin index had to be updated via `kubectl krew update` and the plugin can be installed via `kubectl krew install view-secret`.

Now the workflow to decode secrets is as simple as this:

```bash
# print secret keys
kubectl view-secret <secret>

# decode specific entry
kubectl view-secret <secret> <key>

# decode all secret contents
kubectl view-secret <secret> -a/--all

# print keys for secret in different namespace
kubectl view-secret <secret> -n/--namespace foo

# suppress info output
kubectl view-secret <secret> -q/--quiet
```

This was my first [CNCF][cncf] contribution & I'm happy about the feedback I got from [@ahmetb][ahmetb] & [@corneliusweig][corneliusweig] throughout the process.

The full plugin code is available on [GitHub][view-secret].

Thanks for reading! As always please reach out if you have any questions. :wave:

[kubernetes]: https://kubernetes.io
[krew]: https://krew.dev
[krew-index]: https://github.com/kubernetes-sigs/krew-index
[install]: https://github.com/kubernetes-sigs/krew/#installation
[Base64]: https://en.wikipedia.org/wiki/Base64
[kubedecode]: https://github.com/mveritym/kubedecode
[view-secret-old]: https://github.com/ahmetb/kubectl-extras/tree/master/view-secret
[view-secret]: https://github.com/elsesiy/kubectl-view-secret
[krew-contrib]: https://github.com/kubernetes-sigs/krew-index/pull/287
[krew-doc]: https://github.com/kubernetes-sigs/krew/blob/master/docs/DEVELOPER_GUIDE.md
[go-releaser]: https://goreleaser.com/
[travis-ci]: https://travis-ci.com/
[previous]: /blog/containerization-of-golang-applications
[cobra]: https://github.com/spf13/cobra
[k8s-client-go]: https://github.com/kubernetes/client-go
[k8s-cli-runtime]: https://github.com/kubernetes/cli-runtime
[ahmetb]: https://github.com/ahmetb
[corneliusweig]: https://github.com/corneliusweig
[cncf]: https://www.cncf.io/

---
layout: post
title:  "Kubernetes: oauth2_proxy with dynamic callback urls"
categories: []
tags:
- oauth2_proxy
- nginx
- sso
- single sign-on
- reverse proxy
- nginx
- kubernetes
status: publish
type: post
published: true
---
We all love the simplicity of deploying applications on Kubernetes and while many tutorials out there help you get started quickly and provide a great resource for many, some of them spare important details. In this post, I try to [help][help] the community by providing a small guide on how to deploy [oauth2_proxy][oauth2_proxy] with dynamic callback urls. But first, what is oauth2_proxy and which problem does it solve?

The `README.md` explains it as follows:

    A reverse proxy and static file server that provides authentication using Providers (Google, GitHub, and others) to validate accounts by email, domain or group.

Okay great, so this tool comes in handy if you want to authenticate users for an application that doesn't offer authentication by itself. Famous examples include the [Kubernetes dashboard][kubernetes-dashboard], [Prometheus][prometheus] or [Alertmanager][alertmanager].

There are multiple ways to solve the issue of serving apps that don't offer authentication out-of-the-box:

1. Don't expose it at all and just browse it using `kubectl port-forward`. This way, the application is never publicly exposed on the internet.
2. Expose it and handle authentication in a proxy sitting in front of the application using `oauth2_proxy` via existing providers (Microsoft, GitHub, etc.).
3. Establish your own (federated) idendity provider to handle user authentication using i.e. [dex][dex].

<!--more-->
For the first option everyone who needs access to these tools need cluster access, so this is not a very flexible option. The second option is definitely more interesting because we can safely expose the applications on the public internet without much effort. The third option offer the most flexiblity but is a bit of an overkill for what I'm trying to achieve. Hence, I'll be focusing on No. 2. But we might not want to have mulitple proxies in place which handle authentication independently for the respective app but rather a single instance that can be used by all apps. Let's go through an example:

I want to expose Alertmananger and Prometheus to the same group of people and they should be able to seemlessly switch between the applications without the need to sign-in again.

First, I'll be using helm to install the chart for `oauth2_proxy` and setting some custom properties which I need for the OAuth2 provider of my choice:

    helm install stable/oauth2-proxy --name login-oauth2-proxy \
    --namespace xyz \
    --set config.clientID="${CLIENT_ID}" \
    --set config.clientSecret="${CLIENT_SECRET}" \
    --set config.cookieSecret="${COOKIE_SECRET}" \
    --set extraArgs.provider="azure" \
    --set extraArgs.azure-tenant="${AZURE_TENANT_ID}" \
    --set extraArgs.whitelist-domain=".mydomain.com" \
    --set extraArgs.cookie-domain=".mydomain.com" \
    --tls

`clientID`, `clientSecret` and `azure-tenant` can be obtained upon registering the application for Azure Active Directory integration as described [here][azure-ad-app-registration]. To restrict access to only a subset of users from the Active Directory make sure to follow [these instructions][azure-ad-restrict-access] after. It's important to register the url of the ingress rule that will be used for authentication (see below), in my case `https://login.mydomain.com/oauth2/callback`.

The `cookieSecret` is just a random secret that can be generated with a simple python script.

    docker run -ti --rm python:3-alpine python -c 'import secrets,base64; print(base64.b64encode(base64.b64encode(secrets.token_bytes(16))));'

Worth mentioning are the `whilelist-domain` and `cookie-domain` flags which should point to the parent domain of the applications to be protected, i.e.  
You want to protect `prom.mydomain.com` and `alerts.mydomain.com` then this needs to be set to `.mydomain.com`.

Great, now we need an ingress route that handles authentication via the proxy we just deployed. This looks as simple as this:

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: login-ingress-oauth2
      namespace: xyz
      annotations:
        kubernetes.io/ingress.class: nginx
    spec:
      rules:
      - host: login.mydomain.com
        http:
          paths:
          - backend:
              serviceName: login-oauth2-proxy
              servicePort: 80
            path: /oauth2
      tls:
      - hosts:
        - login.mydomain.com

Now all we have to do for the application that should be protected via our proxy is to use the `auth-url` and `auth-signin` annotations of nginx and have them reference this ingress route.

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: alertmanager-ingress
      namespace: tiller
      annotations:
        kubernetes.io/ingress.class: nginx
        nginx.ingress.kubernetes.io/auth-url: "https://login.mydomain.com/oauth2/auth"
        nginx.ingress.kubernetes.io/auth-signin: "https://login.mydomain.com/oauth2/start?rd=https://$host$request_uri"
    spec:
      rules:
      - host: alerts.mydomain.com
        http:
          paths:
          - backend:
              serviceName: prom-prometheus-operator-alertmanager
              servicePort: 9093
            path: /
      tls:
      - hosts:
        - alerts.mydomain.com

Browsing `alerts.mydomain.com` will redirect to the microsoft login and after successful authentication back to the application. If you deploy multiple application using this method you won't have to login again as the consent has been granted already and a valid cookie exists.

A few things to be mentioned:

1. Depending on how many applications rely on the proxy, you might want to scale the oauth2_proxy deployment to ensure availability
2. None of the explanations above indicate that you shouldn't be taking care of proper RBAC rules in your cluster and restrict access to the applications according to the principle of least privilege.

That's it for now, please reach out if you have further questions or remarks. Happy Sunday!

[help]: https://github.com/pusher/oauth2_proxy/issues/109
[oauth2_proxy]: https://github.com/pusher/oauth2_proxy
[kubernetes-dashboard]: https://github.com/kubernetes/dashboard
[prometheus]: https://github.com/prometheus/prometheus
[alertmanager]: https://github.com/prometheus/alertmanager
[dex]: https://github.com/dexidp/dex
[azure-ad-app-registration]: https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application
[azure-ad-restrict-access]: https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-restrict-your-app-to-a-set-of-users

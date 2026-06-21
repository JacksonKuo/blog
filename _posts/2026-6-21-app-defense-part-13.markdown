---
layout: post
title: "Application Defense: Part XI - Okta"
date: 2026-6-20
tags: ["app"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Build a bunch of application defensive mechanisms. This problem can be further broken down into:

* Step 1: Build a sample app with Spring Boot
* Step 2: Build a deployment pipeline
* Step 3: Add hCaptcha
* Step 4: Upgrade deployment pipeline with K8s
* Step 5: Add custom rate limit
* Step 6: Add Twilio MFA
* Step 7: Add Smokescreen
* Step 8: Add test suite
* Step 9: Add test automation
* Step 10: Add Dependabot
* Step 11: **Add Okta**

# Index
How to add Okta integration to an application. This includes the following sections:

1. Okta Admin Account
2. Okta Documentation
3. Deploy Gin Server
4. Okta Configuration

# Okta Admin Account
The Okta Platform has the Okta Integrator Plan that can be used to test Okta integration before deploying to production.[^1] This is the preferred method for testing as the Okta Developer Edition is now deprecated.[^2] [^3] And it does require a work email account.

Digging up my old Okta accounts:
* Okta Developer accounts
    * (deactivated)
* Okta Integrator account
    * [https://integrator-4382332.okta.com/](https://integrator-4382332.okta.com/)

# Okta Documentation
There's a lot of documentation, which can get a little confusing. Okta has two deployment models:[^4]
* Redirect authentication / Okta-hosted
* Embedded authentication / Self-hosted

In their "Choose your Auth" section, Okta recommends using redirect authentication for most cases including server-side, SPA, and mobile.[^5] There's a quickstart guide for various languages.[^6] The golang guide is here [https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/](https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/).

And here is a list of officially supported resources and their GitHub orgs:
* [Recommended Okta SDKs]([https://developer.okta.com/code)
* [Recommended Okta SDKs - server-side web apps](https://developer.okta.com/code/#for-server-side-web-apps)
* [GitHub Org - okta](https://github.com/orgs/okta/repositories)
* [GitHub Org - okta-samples](https://github.com/orgs/okta-samples/repositories)

| Language | Org | Repo | Status |
|---|---|---|---|
| Golang | okta | [samples-golang](https://github.com/okta/samples-golang/tree/master/okta-hosted-login) | archived | 
| Golang | okta | okta-jwt-verifier-golang | active | 
| Golang | okta-sample | [okta-go-gin-sample](https://github.com/okta-samples/okta-go-gin-sample) | active | 
| Golang | okta-sample | [okta-go-api-sample](https://github.com/okta-samples/okta-go-api-sample) | active | 
| Python | okta | samples-python-flask | archived | 
| Python | okta | okta-jwt-verifier-python | active | 
| React | okta | okta-auth-js | active | 
| React | okta | okta-jwt-verifier-js | active | 

# Deploy Gin Server
Alright, time to deploy a gin server. Alternatively, I could run this on localhost[^9]

I follow the same pattern as the `app-smokescreen` repo. [https://github.com/JacksonKuo/app-gin](https://github.com/JacksonKuo/app-gin) builds a package on the public repo, which is used by the `app-springboot` pipeline helm charts. One day I should refactor the name of `app-springboot`. 

```shell
go mod init github.com/JacksonKuo/app-gin
go get github.com/gin-gonic/gin
go mod tidy
```

I use the sample code from [https://github.com/gin-gonic/gin](https://github.com/gin-gonic/gin) for my gin application. I do have to set the port to `2096` as Cloudflare only proxies certain ports for SSL.[^10] And for SSL I can just volumeMount again and pass the PEMs to `runTLS`.  

```golang
certFile := "/etc/letsencrypt/live/bakacore.com/fullchain.pem"
keyFile := "/etc/letsencrypt/live/bakacore.com/privkey.pem"

  var err error
  if _, statErr := os.Stat(certFile); statErr == nil {
    log.Println("TLS cert found; serving HTTPS on :2096")
    err = r.RunTLS(":2096", certFile, keyFile)
  } else {
    log.Println("no TLS cert; serving HTTP on :2096")
    err = r.Run(":2096")
```

My request [https://bakacore.com:2096/ping](https://bakacore.com:2096/ping) returns `{"message":"pong"}`.

# Okta Configuration
And now I just follow these instructions to setup Okta Test Redirect Authentication:
* [https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/](https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/)
* [https://github.com/okta-samples/okta-go-gin-sample](https://github.com/okta-samples/okta-go-gin-sample)

We want to use `okta-go-gin-sample` as a ref over `samples-golang` because gin is a proper framework that can handle auth. 



# Reference
[^1]: [https://developer.okta.com/signup/](https://developer.okta.com/signup/)
[^2]: [https://developer.okta.com/blog/2025/05/13/okta-developer-edition-changes](https://developer.okta.com/blog/2025/05/13/okta-developer-edition-changes)
[^3]: [https://support.okta.com/help/s/article/introducing-integrator-free-orgs](https://support.okta.com/help/s/article/introducing-integrator-free-orgs)
[^4]: [https://developer.okta.com/docs/concepts/redirect-vs-embedded/](https://developer.okta.com/docs/concepts/redirect-vs-embedded/)
[^5]: [https://developer.okta.com/docs/guides/sign-in-overview/main/#choose-your-auth](https://developer.okta.com/docs/guides/sign-in-overview/main/#choose-your-auth)
[^6]: [https://developer.okta.com/docs/guides/quickstart/main/#server-side-web-app](https://developer.okta.com/docs/guides/quickstart/main/#server-side-web-app)
[^7]: [https://developer.okta.com/code/](https://developer.okta.com/code/)
[^8]: [https://github.com/orgs/okta/repositories?q=sample](https://github.com/orgs/okta/repositories?q=sample)
[^9]: [https://developer.okta.com/blog/2021/01/04/offline-jwt-validation-with-go](https://developer.okta.com/blog/2021/01/04/offline-jwt-validation-with-go)
[^10]: [https://developers.cloudflare.com/fundamentals/reference/network-ports/#network-ports-compatible-with-cloudflares-proxy](https://developers.cloudflare.com/fundamentals/reference/network-ports/#network-ports-compatible-with-cloudflares-proxy)
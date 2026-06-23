---
layout: post
title: "Application Defense: Part XI - Okta"
date: 2026-6-22
tags: ["app"]
published: true
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
* Step 11: **Add Okta integration**

# Okta Admin Account
The Okta Platform has the Okta Integrator Plan that can be used to test Okta integration before deploying to production.[^1] This is the preferred method for testing as the Okta Developer Edition is now deprecated.[^2] [^3] And it does require a work email account.

Digging up my old Okta accounts:
* Okta Developer accounts
    * (deactivated)
* Okta Integrator account
    * [https://integrator-4382332.okta.com](https://integrator-4382332.okta.com)

# Okta Documentation
There's a lot of documentation, which can get a little confusing. Okta has two deployment models:[^4]
* Redirect authentication / Okta-hosted
* Embedded authentication / Self-hosted

In their "Choose your Auth" section, Okta recommends using redirect authentication for most cases including server-side, SPA, and mobile.[^5] There's a quickstart guide for various languages.[^6] The golang guide is here [https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/](https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/).

And here is a list of officially supported resources and their GitHub orgs:
* [Recommended Okta SDKs](https://developer.okta.com/code)
* [Recommended Okta SDKs - server-side web apps](https://developer.okta.com/code/#for-server-side-web-apps)
* [GitHub Org - okta](https://github.com/orgs/okta/repositories)
* [GitHub Org - okta-samples](https://github.com/orgs/okta-samples/repositories)

| Language | Org | Repo | Status |
|---|---|---|---|
| Golang | okta | [samples-golang](https://github.com/okta/samples-golang/tree/master/okta-hosted-login) | archived | 
| Golang | okta | okta-jwt-verifier-golang | active | 
| Golang | okta-samples | [okta-go-gin-sample](https://github.com/okta-samples/okta-go-gin-sample) | active |
| Golang | okta-samples | [okta-go-api-sample](https://github.com/okta-samples/okta-go-api-sample) | active |
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

My request to [https://bakacore.com:2096/ping](https://bakacore.com:2096/ping) returns `{"message":"pong"}`.

# Okta Configuration
And now I just follow these instructions to setup Okta Test Redirect Authentication:
* [https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/](https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/)
* [https://github.com/okta-samples/okta-go-gin-sample](https://github.com/okta-samples/okta-go-gin-sample)

We want to use `okta-go-gin-sample` as a ref over `samples-golang` because gin is a proper framework that can handle auth. 

Instructions
* Admin Console = [https://integrator-4382332.okta.com](https://integrator-4382332.okta.com)
    * Applications > Applications
    * Create App Integration
        * Sign-in method = `OIDC - OpenID Connect`
        * Application type = `Web Application`
        * App integration name = `GoApp`
        * Sign-in redirect URIs = [https://bakacore.com:2096/authorization-code/callback](https://bakacore.com:2096/authorization-code/callback)
        * Sign-out redirect URIs = [https://bakacore.com:2096](https://bakacore.com:2096)
        * Controlled access = `Allow everyone in your organization to access`
    * Keep track of these values
        * Client ID
        * Client Secret
        * Security > API > Issuer URI
        * Session Key

And save them as the following GitHub Action Secrets, which are used by the workflow and injected into the k8s secrets and then injected into env variables:
* `OKTA_OAUTH2_ISSUER`
* `OKTA_OAUTH2_CLIENT_ID`
* `OKTA_OAUTH2_CLIENT_SECRET`
* `OKTA_OAUTH2_SESSION_KEY`

Non-sensitive values can be added directly in the `gin-deployment.yaml`
* `OKTA_REDIRECT_URI`

My request to [https://bakacore.com:2096/env](https://bakacore.com:2096/env) returns:

```json
{"OKTA_OAUTH2_ISSUER":"https://integrator-4382332.okta.com/oauth2/default","OKTA_REDIRECT_URI":"https://bakacore.com:2096/authorization-code/callback"}
```

# Okta App Integration

#### Architecture Stack
Some languages + frameworks have plug-and-play middleware Okta integration code, such as:
* Java/Spring Boot
* .NET
* Node/Express

But for Golang that's not the case. There is some example code:
1. [okta-go-gin-sample](https://github.com/okta-samples/okta-go-gin-sample)
    * `golang.org/x/oauth2`
    * `gorilla/sessions`
    * `okta-jwt-verifier-golang`
2. [sign-into-web-app-redirect](https://developer.okta.com/docs/guides/sign-into-web-app-redirect/go/main/)
    * `net/http`
    * `gorilla/sessions`
    * `okta-jwt-verifier-golang`

So the architecture stack options seem to be the following:

| Stack | Component |
|---|---|
| Web | `net/http`<br>`gin-gonic/gin` | 
| OAuth | `golang.org/x/oauth2` | 
| Sessions | `gorilla/sessions`<br>`gin-contrib/sessions` | 
| Verifier | `okta-jwt-verifier-golang`<br>`coreos/go-oidc`  | 
| Middleware | `oauth2-proxy`[^11] |

#### Okta Integration Components
And then for the test redirect, the essential test redirect components are:

* environment variable setup
* handlers
    * `/login`
    * `/logout`
	* `/authorization-code/callback`
        * `exchangeCode()`
        * `verifyToken()`
	* `/profile`
* authn/authz
	* `isAuthenticated()`
	* `middleware()`

I updated the `main.go` based on the test redirect code in [https://github.com/JacksonKuo/app-gin](https://github.com/JacksonKuo/app-gin). Reran `kubectl rollout restart deployment app-gin`. 

#### Okta Access Setup
* Disable 
    * General > Proof of Possession
    * General > Federation Broker Mode
* Create
    * Directory > People > Add Person
    * Directory > Group > App Group > `GoApp`
        * Assign People
    * Application > Assignments > Assign > Assign to Groups > `GoApp`
    * Security > API > Access Policies
        * New Access Policies > `GoAppPolicy`
        * Add Rule > Grant > click `Authorization Code` > Create rule

Lastly login via [https://bakacore.com:2096/login](https://bakacore.com:2096/login). Response is 

```json
{"email":"redact@gmail.com","email_verified":true,"family_name":"K","given_name":"J","locale":"en_US","name":"J K","preferred_username":"redact@gmail.com","sub":"00u14fsx0z10TCgzJ698","updated_at":1782189241,"zoneinfo":"America/Los_Angeles"}
```

This is just a quick and dirty PoC to get things working.

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
[^11]: [https://developer.okta.com/blog/2022/07/14/add-auth-to-any-app-with-oauth2-proxy](https://developer.okta.com/blog/2022/07/14/add-auth-to-any-app-with-oauth2-proxy)
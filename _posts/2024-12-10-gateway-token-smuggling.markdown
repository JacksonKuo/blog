---
layout: post
title: "Gateway Token Smuggling - A Primer"
date: 2024-12-10
tags: ["vulnerability"]
---

# Problem Statement

Describe the Gateway Token Smuggling technique.[^1]

# Introduction

This technique is something I call Gateway Token Smuggling and involves smuggling tokens past an API gateway in order to impersonate any account. I found an instance of this vulnerability in Kong Gateway 3.2.1.0.[^2]

![Image]({{ site.baseurl }}/assets/images/token-smuggling.png)

A couple of important points. 

* API gateways will often accept a credential token from one of three search locations: 
    1. query parameters
    2. cookies
    3. http headers
* Since all requests must flow through the API gateway and requests are only forwarded if the gateway has authenticated the credential token, developers may elect to have backend services implicitly trust that the token is valid. Developers may choose for the backend service to only perform authorization checks on JWT claims, with the assumption that authentication has already been performed by the gateway
* Having backend services implicitly trusting the token intuitively makes sense, since performing the same authentication check at the gateway and application layer results in duplicate work
* Kong Gateway parses tokens in the following order:[^3]
    1. `jwt` query parameter
    2. `jwt` cookie
    3.  `Authorization: Bearer` header
* Lastly, if any token location contained a valid token, Kong Gateway will forward the entire request to the backend service

# Vulnerability

Maybe you've already spotted how this pattern can go wrong. What if the following application and environment exists?
* A backend service accepts a token from the `Authorization: Bearer` header. This pattern is by far the most common token location
* The backend service trusts the token has been authenticated by the gateway and only performs authorization checks on the JWT claims

What if an attacker submits the following request with two tokens. One token is legitimate and signed in the query parameter. The other token is fake and not signed?

```
GET /user?jwt=[VALID JWT]
Host: internal.service
Authorization: Bearer: base64({"typ": "JWT"}.{"admin": true}.{})
```

1. The above request is sent to the API gateway, with a valid JWT in the `jwt` query parameter and a malicious JWT in the `Authorization: Bearer` header. Note the query parameter contains a valid JWT. The header contains a spoofed JWT that is not signed 
2. The Kong Gateway parses the `jwt` parameter first, finds that the JWT signature is valid, then forwards the full request to the backend service
3. The backend service ignores the `jwt` query parameter, only parses the JWT in the header, reads the JWT claims, and then performs any authorization checks based on the parsed claims

As a result, an attacker can mint a JWT with arbitary token claims, smuggling the token through the gateway to a backend service, and impersonate any account. The default JWT plugin for Kong gateway was vulnerable to this issue. 

# Mitigation

* All token search locations should be validated and all tokens must be the same, which is how Kong mitigated this issue:[^4]

> If Kong finds multiple tokens that differ - even if they are valid - the request will be rejected to prevent JWT smuggling.

* Backend services should reauthenticate requests. While Kong states:

> The JWT will be forwarded to your upstream service, which can assume its validity. 

I would disagree with this validity assumption. While revalidating tokens at both the gateway and service layers seems unnecessary and is a duplication of work, revalidating is the safer practice and follows a zero-trust policy. Backend services should never assume tokens have been validated. Revalidation prevents attack vectors such as: gateway token smuggling, SSRF attacks, http request smuggling, http desync attacks, and compromised internal networks. 

**The architecture of gateways performing authentication and backend services only performing authorization is a insecure design pattern.**

An argument could be made of why even use a API gateway if the endpoint still has to revalidate the token, which is fair a point. But a better way to think of the initial validation is that you need a valid token to be able to initiate communication to a backend service. If we imagine an office building, the first authentication check gets you through the door, but each floor requires a second authentication check as well as an authorization check.

# References
[^1]: Note there exists a similiarly named "token smuggling" term in the LLM space, which is why i prefix my technique with the title "gateway" to differentiate. Though anything that forwards traffic like proxies, middleware, authorizers, or service meshes has potential to run afoul of token smuggling.
[^2]: [https://docs.konghq.com/gateway/changelog/#3210](https://docs.konghq.com/gateway/changelog/#3210)
[^3]: [https://docs.konghq.com/hub/kong-inc/jwt/#send-a-request-with-the-jwt](https://docs.konghq.com/hub/kong-inc/jwt/#send-a-request-with-the-jwt)
[^4]: [https://docs.konghq.com/hub/kong-inc/jwt/](https://docs.konghq.com/hub/kong-inc/jwt/)

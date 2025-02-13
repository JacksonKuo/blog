---
layout: post
title: "Application Defense: Part VII - Smokescreen"
date: 2025-2-13
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
* Step 4: Upgrade deployment pipeline
* Step 5: Add custom rate limit
* Step 6: Add Twilio MFA
* Step 7: **Add Smokescreen**

# Thought Process
The next application defense to add Smokescreen for SSRF defense. I think I've done this in the past, but good to have it on hand.

There a number of ways to deploy smokescreen in a k8s cluster.

* Deployment + Service
* DaemonSet[^1]
* Sidecar

The ideal setup is for smokescreen to be a callable service by other services.  DaemonSet would create an instance on each node, but I don't necessary need smokescreen on every node. I also don't need smokescreen as a sidecar, since other services can't call the sidecar. So deployment + service makes sense to me.

In a perfect world, I woudl create K8 NetworkPolicies to restrict all internet access, and the proxy everything through smokescreen.

However, in practice that seems like a bad idea. I really don't want to manage ACLs for every single service and web request in a company. Instead, the applications that make outbound calls with user-provided URLs should proxy their http client to smokescreen.

As for authentication, the default mTLS authentication seems a bit too unwieldy. Fly.io uses a proxy password and if you're deny-listing all traffic then that makes sense.[^2] But if you're allow-listing all traffic then authentication doesn't really matter. 

# Smokescreen
Smokescreen is built into a image and saved into ghcr.io. Helm charts then pull the image. Smokescreen 

* Endpoint: [https://bakacore.com:8087/smokescreen](https://bakacore.com:8087/smokescreen)
* Repository: [https://github.com/JacksonKuo/app-smokescreen](https://github.com/JacksonKuo/app-smokescreen)
* Service: [https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/controller/SmokescreenController.java](https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/controller/SmokescreenController.java)
* Controller: [https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/service/SmokescreenService.java](https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/service/SmokescreenService.java)

#### Application
I'm using Spring Webflux + Netty to make my webclient. This is a bit different than the typical Spring MVC + Tomcat setup, and uses things like `Mono<String>` to enable non-blocking web requests.

#### Config
Running the default smokescreen `./smokescreen` will only prevent access to internal networks. Also by default, smokescreen also tries to use mTLS for client authentication. In order to skip mTLS client validation, `allow_missing_role: true` is set in `config.yaml`. With this set, smokescreen will then use the default ACL.[^3].

Smokescreen uses a HTTP CONNECT proxy that accepts HTTP and HTTPS. 

#### Oddities
I don't see a way to set an IP address in the `acl.yaml` file. So if I need allow an internal IP address, the following works. 

`./smokescreen --allow-address 127.0.0.1`

There is a `--unsafe-allow-private-ranges` flag but it isn't required to deny/allow an IP address. 

When my Spring Boot app proxies HTTPS requests, I get an odd error back: `failure when writing TLS control frames`. Smokescreen still works as intended, but I don't get the expected: `Failed to fetch data: http, none, smokescreen/10.43.253.119:4750 => example.org/<unresolved>:80, status: 407 Request rejected by proxy`

I'm not 100% sure how smokescreen affects DNS rebinding. 

# References
[^1]: *In order to keep things performant, we run Smokescreen in a DaemonSet so each node has its own instance. The applications running on that node contact the node’s IP on a known port in order to keep traffic local.* [https://material.security/resources/locking-down-internet-traffic-in-kubernetes](https://material.security/resources/locking-down-internet-traffic-in-kubernetes)

[^2]: [https://fly.io/blog/practical-smokescreen-sanitizing-your-outbound-web-requests/](https://fly.io/blog/practical-smokescreen-sanitizing-your-outbound-web-requests/), [https://fly.io/docs/app-guides/smokescreen/](https://fly.io/docs/app-guides/smokescreen/)

[^3]: [https://github.com/stripe/smokescreen/blob/master/Development.md](https://github.com/stripe/smokescreen/blob/master/Development.md)
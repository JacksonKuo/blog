---
layout: post
title: "Application Defense: Part XI - Okta"
date: 2026-6-21
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

# Okta
How to add Okta integration to an application. This includes the following steps:

1. Setup Okta Admin
2. Integrate Okta with App

# Okta Admin
The Okta Platform has the Okta Integrator Plan that can be used to test Okta integration before deploying to production.[^1] This is the preferred method for testing as the Okta Developer Edition is now deprecated.[^2] [^3] And it does require a work email account.

I need to dig up my old Okta accounts. My dev accounts are activated.

* Integrator Account
    * [https://integrator-4382332.okta.com/](https://integrator-4382332.okta.com/)
    * Baka org

# Okta Integration

https://developer.okta.com/blog/2021/01/04/offline-jwt-validation-with-go

https://developer.okta.com/docs/guides/redirect-authentication/

https://github.com/okta/okta-jwt-verifier-golang


# Reference

[^1]: [https://developer.okta.com/signup/](https://developer.okta.com/signup/)

[^2]: [https://developer.okta.com/blog/2025/05/13/okta-developer-edition-changes](https://developer.okta.com/blog/2025/05/13/okta-developer-edition-changes)

[^3]: [https://support.okta.com/help/s/article/introducing-integrator-free-orgs](https://support.okta.com/help/s/article/introducing-integrator-free-orgs)
---
layout: post
title: "Application Defense: Part XII - Socket.dev"
date: 2026-6-23
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
* Step 11: Add Okta integration
* Step 11: **Add Socket.dev**

# Socket.dev Setup
Test Socket.dev labels to apply policy to individual repositories. I basically want to test socket functionality but my repos, before roll them out to the rest of the org

First question is org or personal account? Uh, let's do org:
1. [Sign In](https://socket.dev/auth/login) > Continue with Email
2. [create-organization](https://socket.dev/dashboard/create-organization) > Install > `jkuo-org` > All repositories > Install & Authorize
3. Link installation > `jkuo-org`

* GHApp Permissions
    * Read access to issues, members, merge queues, and metadata
    * Read and write access to checks, code, and pull requests

https://docs.socket.dev/docs/getting-started

Do labels override or supplement the securit policy?

#### PR Status Check

Default Policy: Default

what pops up 

no per org access control? maybe repo access

#### Label Specific Repos
They moved the org settings to under the org name.

https://socket.dev/dashboard/org/jkuo-org/repositories/labels

Labels can have distinct security policies. 

Repositories > Add Labels

#### Alerts

* Slack 
* Webhooks
* Subscriptions

#### New Features
Admin
    Repo Access

# Reference
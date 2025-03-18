---
layout: post
title: "Application Defense: Part XI - Deployment Pipeline 3.0"
date: 2025-3-16
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
* Step 11: *Update deployment pipeline with Terraform**

# Cloud deployment pipeline
Let's add a Terraform module to do the initial Digital Oceans infrastructure setup with the option to be able to swap between different cloud ecosystems.

# Terraform


# Terraform Module
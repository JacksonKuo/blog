---
layout: post
title: "Application Defense: Part VIII - Test Suite"
date: 2025-2-14
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
* Step 4: Upgrade deployment pipeline
* Step 5: Add custom rate limit
* Step 6: Add Twilio MFA
* Step 7: Add Smokescreen
* Step 8: Add test suite

# Test Suite

As typical of Spring Boot apps, I'l be using JUnit and Mockito. There are 4 services that could use tests:

* Rate limit
* Smokescreen
* Twilio
* hcaptcha

# Test Methodology

To help me on thinking about and writing tests, i read through [Unit Testing Principles, Practices, and Patterns](https://www.amazon.com/gp/product/B09782L692/) by Vladimir Khorikov, which was very enlightening. That said, a lot of the books guidance hasn't fully sunk in, so I'm sure my tests and code will still have all sorts of problems. Regardless, let's get to it!

#### Rate limit

The first test is the redis rate limit. 

#### Smokescreen

#### Twilio

#### hcaptcha

# References
[^1]: []()
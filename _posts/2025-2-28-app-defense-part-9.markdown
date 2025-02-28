---
layout: post
title: "Application Defense: Part IX - Test Automation"
date: 2025-2-27
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
* Step 9: **Add test automation**

# Test Automation
There is a couple of upgrades I want to add to testing:

* Code coverage via Jacoco
* Coverage badge on Github
* Test automation via CircleCI

# Code Coverage via Jacoco
Pretty simple. 

```kotlin
plugins {
	...
	jacoco
}
...
tasks.test {
    finalizedBy(tasks.jacocoTestReport) // report is always generated after tests run
}
tasks.jacocoTestReport {
    dependsOn(tasks.test) // tests are required to run before generating the report
	reports {
        xml.required = false
        csv.required = false
        html.outputLocation = layout.buildDirectory.dir("jacocoHtml")
    }
}
```

I did have to upgrade to the latest version of `testImplementation("org.junit.jupiter:junit-jupiter-api")` due to some errors that popped up. Test coverage shows up as 49% currently. 

# Coverage Badge on Github

# Test Automation via CircleCI

# References
[^1]:
---
layout: post
title: "Application Defense: Part IX - Test Automation"
date: 2025-3-2
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
* Step 7: Add Smokescreen
* Step 8: Add test suite
* Step 9: **Add test automation**

# Test Automation
There is a couple of upgrades I want to add to testing:

* Code coverage via Jacoco
* Test automation via CircleCI
* Status badge

# Code coverage via Jacoco
Pretty simple. 

```kotlin
plugins {
    jacoco
}
tasks.test {
    finalizedBy(tasks.jacocoTestReport)
}
tasks.jacocoTestReport {
    dependsOn(tasks.test)
	reports {
        xml.required = false
        csv.required = true
        html.outputLocation = layout.buildDirectory.dir("jacocoHtml")
    }
}
```

I did have to upgrade to the latest version of `testImplementation("org.junit.jupiter:junit-jupiter-api")` due to some errors that popped up. Test coverage shows up as 49% currently. 

# Test automation via CircleCI
I'll be running my automated tests in CircleCI: [https://github.com/JacksonKuo/app-springboot/blob/main/.circleci/config.yml](https://github.com/JacksonKuo/app-springboot/blob/main/.circleci/config.yml)

For the `.gradlew clean test`, the tests required certain env var set, which is why the `application.properties` field have a default set `verify.service.sid=${VERIFY_SERVICE_SID:test}`.

#### Execution environment
Learned a bunch about CircleCI. I'm using Testcontainers which requires a local working docker. And my code also calls docker services on localhost. CircleCI has a bunch of different execution environements defined by executors. 

* docker executor - runs a specific docker image i.e. `cimg/base:stable` but doesn't have docker locally
    * 
        ```
        docker: 
            - image: cimg/base:stable
        ```
* remote docker executor - can run docker commands, but the docker is hosted in a remote location
    * `setup_remote_docker`
* machine - runs ubuntu vms
    * `machine: true`

The only execution environment that easily works out of the box with Testcontainers is machine executors, and then I just install `openjdk-17-jdk`. I could use `setup_remote_docker` and parse the IP address and and then edit my integration tests to reach out to the remote instance, but that's kinda a pain. 

#### Triggers
CircleCI integrates with Github via a Github App. There's webhook that trigger and a status check from CircleCI. Through CircleCI doesn't have as many triggers as I'd hoped:
* `All push`
* `PR opened`
* `PR merged`
* `PR opened or pushed to, default branch pushes, tag pushes`
* `Pushes to open non-draft PRs`

To do PR open and PR push you have to create two triggers, which is kinda annoying.  

#### Performance
CircleCI is pretty nice has provides 30,000 free credits per month. The default machine size is a large ubuntu vm (4CPU, 15 GB), which I've bumped down to `resource_class: medium` (2CPU, 7.5 GB).[^1] [^2]

#### Branch protections
I added some github settings, including some branch protections on `main`:
* `Require a pull request before merging`
* `Require status checks to pass`
    * `ci/circleci: gradle-test`
    * `Do not require status checks on creation`
* `Automatically delete head branches`

#### Miscellaneous
CircleCI status checks automatically link back to the Job, which is nice. And there's a convenience artifact tab for my jacoco HTML report. 

# Status badge
CircleCI has a nice build status badge that can be embedded into the README.[^3]
Using the `style=shield` which looks nicer: [https://github.com/JacksonKuo/app-springboot](https://github.com/JacksonKuo/app-springboot)

# Future enhancements
* Add coverage badge on Github using `jacoco-badge-generator`. I don't see a built-in way to handle coverage badges in CircleCI: [https://github.com/cicirello/jacoco-badge-generator](https://github.com/cicirello/jacoco-badge-generator)
* A nice way to add `jacoco-badge-generator` without clogging up the git history using Github Pages: [https://hackernoon.com/adding-test-coverage-badge-on-github-without-using-third-party-services](https://hackernoon.com/adding-test-coverage-badge-on-github-without-using-third-party-services)
* Play around with Orbs: [https://circleci.com/developer/orbs](https://circleci.com/developer/orbs)
* Terraform for branch protection

# References
[^1]: [https://circleci.com/pricing/price-list/](https://circleci.com/pricing/price-list/)

[^2]: [https://circleci.com/docs/configuration-reference/](https://circleci.com/docs/configuration-reference/)

[^3]: [https://circleci.com/docs/status-badges/](https://circleci.com/docs/status-badges/)
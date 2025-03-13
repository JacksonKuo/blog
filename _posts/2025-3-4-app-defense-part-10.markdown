---
layout: post
title: "Application Defense: Part X - Dependabot"
date: 2025-3-2
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
* Step 9: Add test automation
* Step 9: **Add dependabot**

# Dependabot
Let's add dependabot. Wow dependabot has a lot of documentation. There are 3 main components[^1]:
* Dependabot alerts[^2]
    - *scans default branch when either a new advisory is created in Github Advisory DB[^3] or when the dependency graph changes*
    - *Dependabot alerts rely on the dependency graph*
    - *Dependabot will only create Dependabot alerts for vulnerable GitHub Actions that use semantic versioning.*
* Dependabot security updates: auto PR for security updates
* Dependabot version updates: auto PR for version updates

#### Dependencies
[https://github.com/JacksonKuo/app-springboot/blob/main/.github/dependabot.yml](https://github.com/JacksonKuo/app-springboot/blob/main/.github/dependabot.yml)

So I have dependabot running for github-actions, docker and gradle. The github-action and gradle are on major versions and won't update that often. My gradle deps are an odd mix of pinned versions and no versioning. The gradle plugins for running tasks are picked up by dependabot[^4]. 

* ci-pipeline.yml
    * ```
    uses: actions/checkout@v4
    uses: actions/setup-java@v4
    uses: actions/cache@v4
    ```
* docker
    * ```
    ARG BASE_IMAGE=openjdk:17-jdk-alpine
    FROM ${BASE_IMAGE}
    ```
* gradle
    * plugins
        * ```
        java
	    id("org.springframework.boot") version "3.4.1"
	    id("io.spring.dependency-management") version "1.1.7"
	    jacoco
        ```
    * dependencies
        * ```
        "org.springframework.boot:spring-boot-starter-web"
        "org.springframework.boot:spring-boot-starter"
        "org.springframework:spring-context"
        "org.springframework.boot:spring-boot-starter-webflux"
        "com.google.code.gson:gson"
        "org.redisson:redisson:3.45.0"
        "com.twilio.sdk:twilio:10.6.10"
        "com.googlecode.libphonenumber:libphonenumber:8.13.55"
        "org.junit.jupiter:junit-jupiter-api"
        "org.testcontainers:testcontainers:1.20.6"
        "org.mockito:mockito-core"
        "org.springframework.boot:spring-boot-starter-test"
        "org.junit.platform:junit-platform-launcher"
        ```

#### Version Pinning
Version pinning is great for when updates have breaking changes, security bugs, or when the supply chain is compromised. 

#### Pull request

Release notes
Changelog
Commits
Compatibility 

Auto rebase

#### Problems

Dependabot opens a lot of PRs

Transitive dependencies could break from lockfiles

# References
[^1]: [https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide](https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide)

[^2]: [https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts)

[^3]: [https://github.com/advisories](https://github.com/advisories)

[^4]: [https://github.com/JacksonKuo/app-springboot/pull/11](https://github.com/JacksonKuo/app-springboot/pull/11)

[^5]: []()

[^6]: []()


#### Documentation
* [https://github.com/JacksonKuo/app-springboot/network/updates](https://github.com/JacksonKuo/app-springboot/network/updates)

* [https://docs.github.com/en/code-security/dependabot](https://docs.github.com/en/code-security/dependabot)

* [https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference)
* [https://github.blog/developer-skills/github/video-how-to-run-dependency-audits-with-github-copilot/](https://github.blog/developer-skills/github/video-how-to-run-dependency-audits-with-github-copilot/)

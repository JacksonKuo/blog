---
layout: post
title: "Application Defense: Part X - Dependabot"
date: 2025-3-16
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
* Step 9: Add test automation
* Step 9: **Add Dependabot**

# Dependabot
Let's add dependabot. Wow, Dependabot has a lot of documentation. There are 3 main components[^1]:
* Dependabot alerts[^2] [^3]
* Dependabot security updates: auto PR for security updates
* Dependabot version updates: auto PR for version updates

#### Dependencies
[https://github.com/JacksonKuo/app-springboot/blob/main/.github/dependabot.yml](https://github.com/JacksonKuo/app-springboot/blob/main/.github/dependabot.yml)

So I have Dependabot running for github-actions, docker and gradle. The github-action and gradle are on major versions and won't update that often. My gradle deps are an odd mix of pinned versions and no versioning. The gradle plugins for running tasks are picked up by Dependabot[^4]. 

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

#### Dependency Locking
I added a `gradle.lockfile` via `./gradlew dependencies --write-locks` [^5] [^6]. 

```build.gradle.kts
dependencyLocking {
   lockAllConfigurations()
}
```

However, Dependabot PR will have a failing gradle status check in CircleCI due to the lockfile not being updated. My test automation pipeline will need to update the PR, or I could leverage Renovate instead.  

#### Version Pinning
In production, dependencies should be version pinned. Pinning mitigates breaking changes, security bugs, or when the supply chain is compromised. 

There are some dependencies that are more reliable than others, for example Spring's minor/patches version have a great reputation for being stable and as long as all tests pass should be safe to upgrade. 

However, other dependencies can be much more flaky and even if tests pass, updates could end up breaking functionality. This is because developer tests are targeted at business logic, and not at the underlying framework. Thus tests aren't necessarily a reliable metric on if a dependency upgrade will cause issues.

#### Dependabot/Lockfile Problems
* Dependabot opens a lot of PRs, though there is an option for grouped security updates
* Dependabot only checks transitive dependencies if there's a lockfile[^7], and gradle support was only recently added in 2023[^8]
* Dependabot PR doesn't modify the lockfile
* Transitive dependencies could break from lockfiles
* Dependabot for certain languages like Golang is not recommended[^9]
* Security PRs will only be created for direct dependencies, not transitive dependencies.[^10] Transitive dependencies for dependency submission API uploads will alert, but not create a PR[^11]

#### Updating Lockfiles
What I need is logic so that if the user is `dependabot`, run `./gradlew dependencies --update-locks <dep-name>` then update the PR. If a dev updates a version they'll need to run `--update-locks` manually. 

I'm pretty disappointed this isn't out-of-the-box for Dependabot. Renovate does this automatically. But for now while I play around with Dependabot functionality, I'll turn off lockfile checks. 

#### Security Alerts
Testing alerts using this Spring Webflux vulnerability:
* [https://github.com/advisories/GHSA-c4q5-6c82-3qpw](https://github.com/advisories/GHSA-c4q5-6c82-3qpw)
* [https://github.com/JacksonKuo/app-springboot/security/dependabot](https://github.com/JacksonKuo/app-springboot/security/dependabot)
* `implementation("org.springframework.security:spring-security-web:6.3.3")`

Looks like dependabot doesn't proc on PRs, only on the default branch. If you want to scan on PRs there is a Github Action: [https://github.com/actions/dependency-review-action](https://github.com/actions/dependency-review-action). Dependabot will proc only after the merge to main and then only according to the scan cadence set up. 

# References
[^1]: [https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide](https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide)

[^2]: *scans default branch when either a new advisory is created in Github Advisory DB or when the dependency graph changes... Dependabot alerts rely on the dependency graph... Dependabot will only create Dependabot alerts for vulnerable GitHub Actions that use semantic versioning.*[https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts)

[^3]: [https://github.com/advisories](https://github.com/advisories)

[^4]: [https://github.com/JacksonKuo/app-springboot/pull/11](https://github.com/JacksonKuo/app-springboot/pull/11)

[^5]: [https://docs.gradle.org/current/userguide/dependency_locking.html](https://docs.gradle.org/current/userguide/dependency_locking.html)

[^6]: [https://medium.com/codex/upgrading-dependencies-doesnt-have-to-suck-8d5b2b767d58](https://medium.com/codex/upgrading-dependencies-doesnt-have-to-suck-8d5b2b767d58)

[^7]: [https://finitestate.io/blog/dependabot-failings/](https://finitestate.io/blog/dependabot-failings/)

[^8]: [https://github.blog/changelog/2023-08-24-gradle-support-for-dependabot-security-updates/](https://github.blog/changelog/2023-08-24-gradle-support-for-dependabot-security-updates/)

[^9]: [https://bsky.app/profile/filippo.abyssdomain.expert/post/3lkdasd64vst3](https://bsky.app/profile/filippo.abyssdomain.expert/post/3lkdasd64vst3)

[^10]: *When you upload Gradle dependencies to the dependency graph using the dependency submission API, all project dependencies are uploaded, even transitive dependencies that aren't explicitly mentioned in any dependency file. When an alert is detected in a transitive dependency, Dependabot isn't able to find the vulnerable dependency in the repository, and therefore won't create a security update for that alert.* [https://docs.github.com/en/code-security/dependabot/ecosystems-supported-by-dependabot/supported-ecosystems-and-repositories](https://docs.github.com/en/code-security/dependabot/ecosystems-supported-by-dependabot/supported-ecosystems-and-repositories)

[^11]: [https://github.com/dependabot/dependabot-core/issues/9351](https://github.com/dependabot/dependabot-core/issues/9351)
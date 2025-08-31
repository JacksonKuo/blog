---
layout: post
title: "Chainguard: Part I -  Infrastructure Debt"
date: 2025-8-17
tags: ["chainguard"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's migrate our base images to Chainguard. This problem can be first broken down into:

* Step 1: Repair the local build pipeline

After not running my local cluster build for a while, I reran my Makefile and had errors. Let's fix this shindig...

#### Problem 1 - .zprofile
OpenJDK upgraded to 24. Which breaks my `~/.zprofile`. Apparently, this was a bad idea: `export PATH="/opt/homebrew/Cellar/openjdk/23.0.1/bin:"$PATH`. Changed to `export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"` which is apparently the recommended way.

#### Problem 2 - gradlew
OpenJDK 24 is not compatible with my current version of Gradle 8.11.1, which needs Gradle 8.14 or later.[^1] Let's go up to Gradle 9. Trying to upgrade my `./gradlew` wrapper ran into some weird type issue. No duckin idea what this is.

```bash
FAILURE: Build failed with an exception.

* Where:
Build file '/Users/jacksonkuo/workspace/app-springboot/build.gradle.kts' line: 52

* What went wrong:
Could not create task ':test'.
> Could not create task of type 'Test'.
   > Could not create an instance of type org.gradle.api.internal.tasks.testing.DefaultTestTaskReports.
      > Could not create an instance of type org.gradle.api.reporting.internal.DefaultReportContainer.
         > Type T not present
```

I just had to comment out this section momentarily in my `build.gradle.kts` and then use `.set()` instead of `=` which apparently is the new way to configure in Kotlin DSL:

```bash
tasks.withType<Test> {
	useJUnitPlatform()
}

tasks.test {
    finalizedBy(tasks.jacocoTestReport) // report is always generated after tests run
}
tasks.jacocoTestReport {
    dependsOn(tasks.test) // tests are required to run before generating the report
	reports {
        xml.required.set(false)
        csv.required.set(true)
        html.outputLocation.set(layout.buildDirectory.dir("jacocoHtml"))
    }
}
```

before running: `./gradlew wrapper --gradle-version=9.0.0 --distribution-type=bin`[^2], and then removing my comments. My `gradle-wrapper.properties` looks good: `distributionUrl=https\://services.gradle.org/distributions/gradle-9.0.0-bin.zip`.

#### Problem 3 - Jekyll
My local Jekyll will not automatically catch file updates anymore for some reason. Using `--force_polling` will reach out and check the if any file has changed versus the default update system that uses `listen`[^3] [^4] that was really flaky as more files were added to jekyll.

```bash
bundle exec jekyll serve --unpublished --livereload --force_polling
```

#### Problem 4 - Makefile
My blog post, [Application Defense: Part IV -  Deployment Pipeline 2.0](https://jacksonkuo.github.io/blog/2025/02/04/app-defense-part-4.html) was using a older Makefile... updated. And added the localhost URL for cluster's local service. Build is working now, woohoo!

#### Problem 5 - vCPU
I pushed the changes to my droplet and my server crashed. It might have been because it's being a while since I've restarted, but the CPU graph maxed out and I couldn't SSH in. I resized my droplet from the $6 regular CPU type to the $7 premium AMD CPU. That's a 16.7% additional compute spend...

# References
[^1]: [https://docs.gradle.org/current/userguide/compatibility.html](https://docs.gradle.org/current/userguide/compatibility.html)

[^2]: [https://gradle.org/install/](https://gradle.org/install/)

[^3]: [https://github.com/jekyll/jekyll-watch/blob/master/jekyll-watch.gemspec](https://github.com/jekyll/jekyll-watch/blob/master/jekyll-watch.gemspec)

[^4]: [https://github.com/guard/listen](https://github.com/guard/listen)


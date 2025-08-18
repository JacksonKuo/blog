---
layout: post
title: "Upgrade - Local Pipeline"
date: 2025-8-17
tags: ["upgrades"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
After not running my local build for a while, I attempted to run my local k3 cluster and ran into errors. Let's fix this shit...

#### Problem 1 - .zprofile
OpenJDK upgraded to 24. Which breaks my `~/.zprofile`. Apparently, this was a bad idea: `export PATH="/opt/homebrew/Cellar/openjdk/23.0.1/bin:"$PATH`. Changed to `export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"` which is apparently the recommended way.

#### Problem 2 - gradlew
OpenJDK 24 is not compatible with my current version of Gradle 8.11.1, which needs Gradle 8.14 or later.[^1] Trying to upgrade my `./gradlew` wrapper ran into some weird type issue. No duckin idea what this is.

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

I just had to comment out this section:

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

before running: `./gradlew wrapper --gradle-version=9.0.0 --distribution-type=bin`

#### Problem 3 - jekyll
My local jekyll will not automatically update anymore for some unknown reason. Using `--force_polling` will reach out and check the if any file have changed versus the event-based system that was really flaky as more files were added to jekyll.

```bash
bundle exec jekyll serve --unpublished --livereload --force_polling
```

#### Problem 3 - Make file
My blog post Deployment Pipeline 2.0[^2] was using a older Makefile... updated.

# References
[^1]: [https://docs.gradle.org/current/userguide/compatibility.html](https://docs.gradle.org/current/userguide/compatibility.html)

[^2]: [https://jacksonkuo.github.io/blog/2025/02/04/app-defense-part-4.html](https://jacksonkuo.github.io/blog/2025/02/04/app-defense-part-4.html)
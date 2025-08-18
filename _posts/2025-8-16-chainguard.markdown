---
layout: post
title: "Chainguard Migration"
date: 2025-8-16
tags: ["devsecops"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's migrate our base images to Chainguard.

# Services

* app-springboot, Dockerfile

```bash
ARG BASE_IMAGE=openjdk:17-jdk-alpine
FROM ${BASE_IMAGE}

WORKDIR /app
COPY build/libs/sample-0.0.1-SNAPSHOT.jar /app
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 8443

ENV JAVA_OPTS="-Xms128m -Xmx256m"
ENV SPRING_PROFILE="prod"

ENTRYPOINT ["/entrypoint.sh"]
```

* app-smokescreen, Dockerfile

```bash
FROM golang:1.23.6-alpine3.21

WORKDIR /app
COPY acl.yaml config.yaml .

RUN apk add --no-cache git
RUN git clone https://github.com/stripe/smokescreen.git 
WORKDIR /app/smokescreen
RUN go build

EXPOSE 4750

CMD ["./smokescreen", "--config-file", "/app/config.yaml", "--egress-acl-file", "/app/acl.yaml"]
#CMD ["./smokescreen"]
```
* redis
    - Helm Chart: `image: redis:latest`

* https://edu.chainguard.dev/chainguard/migration/
* https://edu.chainguard.dev/chainguard/migration/migration-guides/
* https://www.chainguard.dev/unchained/how-to-transition-to-secure-container-images-with-new-migration-guides
* https://www.chainguard.dev/unchained/fully-bootstrapping-java-from-source-in-wolfi
* https://edu.chainguard.dev/chainguard/migration/migration-guides/java-images/

* https://images.chainguard.dev/directory/image/gradle/overview

use trivy to scan for vulns and compare

* https://edu.chainguard.dev/chainguard/chainguard-images/staying-secure/working-with-scanners/trivy-tutorial/

brew install trivy

https://trivy.dev/v0.65/docs/advanced/private-registries/

https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry

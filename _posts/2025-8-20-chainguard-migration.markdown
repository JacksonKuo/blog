---
layout: post
title: "Chainguard: Part III - Migration"
date: 2025-8-18
tags: ["chainguard"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's migrate our base images to Chainguard. This problem can be further broken down into:

* Step 1: Repair the local build pipeline
* Step 2: Run Trivy scans
* Step 3: Chainguard migration

#### Services
I got three services:

* app-springboot - Dockerfile
* app-smokescreen - Dockerfile
* redis:latest` - Helm Chart

I got confirmed my local cluster build is working. Let's just see what happens if I update the base image:

```bash
ARG BASE_IMAGE=openjdk:17-jdk-alpine
ARG BASE_IMAGE=cgr.dev/chainguard/jdk:latest-dev
```

#### Permissions
My `make docker` call failed due to Chainguard running as nonroot and `chmod +x` not being allowed. The solution is change the file permission before writing to the image:

```bash
COPY --chmod=755 entrypoint.sh /entrypoint.sh
#RUN chmod +x /entrypoint.sh
```


brew install chainguard-dev/tap/dfc
dfc -v


Wolfi packages typically match what is available in Alpine,
https://edu.chainguard.dev/chainguard/migration/migrating-to-chainguard-images/#migrating-from-alpine-dockerfiles


https://edu.chainguard.dev/chainguard/migration/migrating-to-chainguard-images/
https://edu.chainguard.dev/chainguard/migration/migration-checklist/

```
jacksonkuo@JacksonKuos-MacBook-Air app-smokescreen % cat ./Dockerfile | dfc -
FROM cgr.dev/ORG/go:1.23-dev
USER root

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


# References
[^1]: []()



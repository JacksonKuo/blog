---
layout: post
title: "Chainguard: Part III - Migration"
date: 2025-8-31
tags: ["chainguard"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's migrate our base images to Chainguard. This problem can be further broken down into:

* Step 1: Repair the local build pipeline
* Step 2: Run Trivy scans
* Step 3: Chainguard migration

# Services
I got three services:

* `app-springboot` - Dockerfile
* `app-smokescreen` - Dockerfile
* `redis:latest` - Helm Chart

#### Dockerfile - Springboot
I've confirmed my local cluster build is working. Let's just see what happens if I update the base image:

```bash
ARG BASE_IMAGE=openjdk:17-jdk-alpine
ARG BASE_IMAGE=cgr.dev/chainguard/jdk:latest-dev
```

Note I could have used `dfc` (docker file convert)[^1], but I just manually edited my Dockerfile. But for reference:

```
brew install chainguard-dev/tap/dfc
dfc -v
dfc ./Dockerfile
```

Also here's a bunch of Chainguard documentation. Note that Chainguard uses a Wolfi distribution based image, which Chainguard calls undistro or distro-less in that they're minimal without a package manager.
* [https://edu.chainguard.dev/chainguard/migration/](https://edu.chainguard.dev/chainguard/migration/)
* [https://edu.chainguard.dev/chainguard/migration/migration-guides/](https://edu.chainguard.dev/chainguard/migration/migration-guides/)
* [https://www.chainguard.dev/unchained/how-to-transition-to-secure-container-images-with-new-migration-guides](https://www.chainguard.dev/unchained/how-to-transition-to-secure-container-images-with-new-migration-guides)
* [https://www.chainguard.dev/unchained/fully-bootstrapping-java-from-source-in-wolfi](https://www.chainguard.dev/unchained/fully-bootstrapping-java-from-source-in-wolfi)
* [https://edu.chainguard.dev/chainguard/migration/migration-guides/java-images/](https://edu.chainguard.dev/chainguard/migration/migration-guides/java-images/)
* [https://images.chainguard.dev/directory/image/gradle/overview](https://images.chainguard.dev/directory/image/gradle/overview)
* [https://www.chainguard.dev/unchained/introducing-wolfi-the-first-linux-un-distro-designed-for-securing-the-software-supply-chain](https://www.chainguard.dev/unchained/introducing-wolfi-the-first-linux-un-distro-designed-for-securing-the-software-supply-chain)

My local `make docker` call failed due to `chmod +x` not being allowed because `chmod` doesn't exist. I first set `--chmod=755`, but we should just `chmod +x entrypoint.sh` the local file and GitHub will remember the execute bit is set:

```bash
COPY entrypoint.sh /entrypoint.sh
#COPY --chmod=755 entrypoint.sh /entrypoint.sh
#RUN chmod +x /entrypoint.sh
```

Now it works locally. Great. Let's try pushing this to my live server. 

```bash
Caused by: java.lang.IllegalStateException: Could not load store from '/etc/letsencrypt/live/bakacore.com/keystore.p12
Caused by: java.nio.file.AccessDeniedException: /etc/letsencrypt/live/bakacore.com/keystore.p12
```

I get an access denied. I'm volume mounting my `keystore.p12` file using `hostpath`, so the file permissions my image sees are whatever is set on my server. Chainguard runs as nonroot, my file is only accessible by root resulting in the error. I run a chmod and problem fixed. 

```bash
root@debian-s-1vcpu-1gb-nyc1-01:~# ls -l /etc/letsencrypt/live/bakacore.com/
-rw------- 1 root root 2789 Aug 15 05:25 keystore.p12

chmod 644 /etc/letsencrypt/live/bakacore.com/keystore.p12
```

Great. That works. Turns out my local tests didn't detect the issue because my local instance doesn't run with SSL. 

Sidenote: I am getting some weird VSCode errors that don't seem to matter outside of my IDE, but I don't understand how to fix them. I'll sort it out later...

```bash
[error] FAILURE: Build failed with an exception.

* What went wrong:
org.gradle.api.plugins.internal.DefaultDecoratedConvention
> org.gradle.api.plugins.internal.DefaultDecoratedConvention
```

#### Dockerfile -  Smokescreen
I'm using `FROM cgr.dev/chainguard/go:latest-dev`. `-dev` will include a shell, but also additional tools like `apk`.

```bash
FROM cgr.dev/chainguard/go:latest-dev
#FROM golang:1.23.6-alpine3.21

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

This configuration fails because the Chainguard `go` image uses a specific entrypoint: `/usr/bin/go`.[^2] Remember `ENTRYPOINT` is the primary executable that runs and `CMD` is the arguments which results in `go ./smokescreen`. This bash is invalid since `smokescreen` is a binary. The yaml has been changed to the following:

```bash
ENTRYPOINT ["./smokescreen"]
CMD ["--config-file", "/app/config.yaml", "--egress-acl-file", "/app/acl.yaml"]
```

#### Helm Chart - Redis
The redis, which I'm not even using right now...

```yaml
#springboot-chart > deployment-redis.yaml

spec:
    containers:
    - name: redis
        #image: redis:latest
        image: cgr.dev/chainguard/redis:latest
        command: ["redis-server", "--maxmemory", "50mb"]
        ports:
        - containerPort: 6379
```

# Results
Whew. Everything is up and running.

Lastly when I push a new build my whole cluster is freezing up. I'm having to restart the whole droplet. Looks like it's the CPU going to 100%. Fun. 

# References
[^1]: [https://github.com/chainguard-dev/dfc](https://github.com/chainguard-dev/dfc)

[^2]: [https://github.com/chainguard-images/images/blob/main/images/go/config/main.tf](https://github.com/chainguard-images/images/blob/main/images/go/config/main.tf)

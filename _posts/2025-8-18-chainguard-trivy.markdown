---
layout: post
title: "Chainguard: Part II - Trivy"
date: 2025-8-18
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

#### Download Spring Image from GHCR
> Apparently, I forgot that I made my images public, so none of the following was necessary. 

Fine-grained PAT tokens don't have `package` permission, weirdly.[^1] So only classic PATs and GitHub Apps can read packages.[^2] 

```bash
export TRIVY_PAT=
echo $TRIVY_PAT | docker login ghcr.io -u JacksonKuo --password-stdin
docker pull ghcr.io/jacksonkuo/springboot:latest
docker inspect ghcr.io/jacksonkuo/springboot:latest
```

#### Install Trivy
Let's see how many CVEs my old images had before i upgrade[^3] [^4]. My `app-springboot` uses `BASE_IMAGE=openjdk:17-jdk-alpine`. Note that `Trivy scans for vulnerabilities and exposed secrets by default`.[^4]

```bash
brew install trivy
trivy image ghcr.io/jacksonkuo/springboot:latest
trivy image ghcr.io/jacksonkuo/springboot:latest --scanners vuln,misconfig,secret

INFO	[vuln] Vulnerability scanning is enabled
INFO	[misconfig] Misconfiguration scanning is enabled
INFO	[secret] Secret scanning is enabled
```

#### Scan Results
The results match what dependabot alerts on. Trivy finds the Netty MadeYouReset CVE-2025-55163 and then does not alert on the Testcontainers CVE-2024-25710 as `testImplementation("org.testcontainers:testcontainers:1.21.0")` is only used when running tests. Docker Scout only finds 28 vulns.[^5]
 
```bash
#### Spring Image Result
trivy image ghcr.io/jacksonkuo/springboot:latest
trivy image ghcr.io/jacksonkuo/springboot:latest --scanners vuln,misconfig,secret

#ghcr.io/jacksonkuo/springboot:latest (alpine 3.14.0)
#Total: 49 (UNKNOWN: 0, LOW: 0, MEDIUM: 10, HIGH: 34, CRITICAL: 5)

#Java (jar)
#Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 1, CRITICAL: 0)
#===

#### OpenJDK Alpine Image Result
docker pull --platform=linux/amd64 openjdk:17-jdk-alpine
trivy image openjdk:17-jdk-alpine

#openjdk:17-jdk-alpine (alpine 3.14.0)
#Total: 49 (UNKNOWN: 0, LOW: 0, MEDIUM: 10, HIGH: 34, CRITICAL: 5)

#### Chainguard Results
docker pull cgr.dev/chainguard/gradle
trivy image cgr.dev/chainguard/gradle

#cgr.dev/chainguard/gradle (wolfi 20230201)
#Total: 0
```

Apparently, `openjdk:17-jdk-alpine` used to be an official image, but has been deprecated since late 2022.[^6] [^7]

# References
[^1]: [https://github.com/github/roadmap/issues/558](https://github.com/github/roadmap/issues/558)

[^2]: [https://trivy.dev/v0.65/docs/advanced/private-registries/](https://trivy.dev/v0.65/docs/advanced/private-registries/)

[^3]: [https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

[^4]: [https://edu.chainguard.dev/chainguard/chainguard-images/staying-secure/working-with-scanners/trivy-tutorial/](https://edu.chainguard.dev/chainguard/chainguard-images/staying-secure/working-with-scanners/trivy-tutorial/)

[^5]: [https://hub.docker.com/layers/library/openjdk/17-jdk-alpine/images/sha256-a996cdcc040704ec6badaf5fecf1e144c096e00231a29188596c784bcf858d05](https://hub.docker.com/layers/library/openjdk/17-jdk-alpine/images/sha256-a996cdcc040704ec6badaf5fecf1e144c096e00231a29188596c784bcf858d05)

[^6]: [https://hub.docker.com/\_/openjdk](https://hub.docker.com/\_/openjdk)

[^7]: [https://docs.docker.com/docker-hub/repos/manage/trusted-content/official-images/](https://docs.docker.com/docker-hub/repos/manage/trusted-content/official-images/)
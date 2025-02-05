---
layout: post
title: "Application Defense: Part IV - Deployment Pipeline 2.0"
date: 2025-2-4
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
* Step 4: **Upgrade deployment pipeline**

# Previous Deployment Pipeline

![Image]({{ site.baseurl }}/assets/images/pipeline-1.0-local.png)

![Image]({{ site.baseurl }}/assets/images/pipeline-1.0-prod.png)

# Upgrade Deployment Pipeline

* [https://github.com/JacksonKuo/app-springboot/releases/tag/v2.0.0](https://github.com/JacksonKuo/app-springboot/releases/tag/v2.0.0)
* [https://github.com/JacksonKuo/app-springboot/blob/main/.github/workflows/ci-pipeline.yml](https://github.com/JacksonKuo/app-springboot/blob/main/.github/workflows/ci-pipeline.yml)

The next app defense upgrade will require running multiple services. The current pipeline with running services using shell scripts and systemd is fine for a simple app, but I'd like to gravitate toward docker and k8s. The problem with docker and especially k8s is the additional memory requirement. 

Currently I have 512 MB of RAM for $4 a month. Me being a penny pincher, I might upgrade to $6 for 1 GB of RAM, but I'm reluctant to spent much more. With only 1 GB of RAM, k8s will likely fail while trying to run other services. 

As for running on other platforms, services like AWS have more provenance, but have a much more complex cost structure and I don't want to run lots of infra in AWS for a long time due to fear of ballooning costs. Other platforms like fly.io just don't have enough learning upside, and is seldom seen in the industry. Using docker-compose.yml has a smaller footprint than k8s, but is also infrequently used in the industry versus just running kubernetes.

To address the RAM limitations, I'm be using K3s:
* Local environment (macOS): k3d
* Droplet environment: K3s

![Image]({{ site.baseurl }}/assets/images/pipeline-2.0-local.png)

![Image]({{ site.baseurl }}/assets/images/pipeline-2.0-prod.png)

# Local Environment Setup: k3d[^1] [^2] [^3]

```bash
brew install k3d
k3d version
k3d cluster create local-cluster --port 8087:8097@loadbalancer
k3d image import springboot --cluster local-cluster
docker exec k3d-local-cluster-server-0 crictl images
alias k="kubectl"
kubectl apply -f deployment_local.yaml

kubectl delete -f deployment_local.yaml
k3d cluster delete local-cluster
```

# Droplet Environment Setup: K3s

I upgraded to a droplet with 1 GB of RAM for $6 per month. And I've moved from systemd to K8s, Helm, and Docker. The Helm package is saved as a Github Release attachment. Docker image is saved to the Github Container Registry. Public container images on `ghcr.io` don't require authentication.[^4] Secret management is handle the same way as pipeline 1.0. 

#### TLS

Certbot is manually run on droplet and then TLS PKCS#12 is volume mounted to the containers. 

* Pipeline 1.0 - Local
  * No TLS
* Pipeline 1.0 - Prod
  * Manual certbot + systemd
* Pipeline 2.0 - Local 
  * No TLS
* Pipeline 2.0 - Prod 
  * Manual certbot + volumeMount

#### Memory Optimization

* Minimal K3s
  * `k3s server --disable traefik --disable servicelb --disable metrics-server`
* Java Heap
  * `CMD ["java", "-jar", "-Xms128m", "-Xmx256m", "sample-0.0.1-SNAPSHOT.jar", "--spring.profiles.active=prod"]`
    * `-Xms128m` = eXtension memory starting, initial minimum heap size
    * `-Xmx256m` = eXtension memory max, maximum heap size
* Add swap space
  * ```fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
```
* Base image size
  * `openjdk:17-jdk-slim` -> `openjdk:17-jdk-alpine`

I currently have around 213 MB and 1 GB of swap space, we'll see how many services i can run before K3s crashes. 

```
free --mega

| Type  | Total | Used | Free | Shared | BuffCache | Avail |
|-------|-------|------|------|--------|-----------|-------|
| Mem   | 1007  | 793  |  80  |   3    |    287    |  213  |
| Swap  | 1073  | 125  | 948  |   —    |     —     |   —   |
```

# References

[^1]: [https://www.perdian.de/blog/2023/07/06/running-kubernetes-locally-on-a-mac-with-k3d/](https://www.perdian.de/blog/2023/07/06/running-kubernetes-locally-on-a-mac-with-k3d/)

[^2]: The `~/.kube/config` created points to `host.docker.internal` which does not resolve correctly. Short fix is to change the address to `localhost`. Long-term fix is to edit `/etc/host` with `127.0.0.1 host.docker.internal`

[^3]: Alternative to using loadbalancer is port-forwarding: `kubectl port-forward svc/springboot-service 8087:8087`

[^4]: [https://github.com/JacksonKuo/app-springboot/pkgs/container/springboot](https://github.com/JacksonKuo/app-springboot/pkgs/container/springboot)
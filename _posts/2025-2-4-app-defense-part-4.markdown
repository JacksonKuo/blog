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

As for running on other platforms, services like AWS have more provenance, but have a much more complex cost structure and I don't want to run lots of infra in AWS for a long time due to fear of ballooning costs. Other platforms like fly.io just don't have enough learning upside and is seldom seen in the industry. Using docker-compose.yml has a smaller footprint than k8s, but is also infrequently used in the industry versus just running kubernetes.

To address the RAM limitations, I'm using K3s:
* Local environment (macOS): k3d
* Droplet environment: K3s

![Image]({{ site.baseurl }}/assets/images/pipeline-2.0-local.png)

![Image]({{ site.baseurl }}/assets/images/pipeline-2.0-prod.png)

#### Environments

I have three environments that i try to maintain: local JAR, local k3d, prod K3s. The defaults will all be for set to prod, i.e. Helm chart `values.yaml`. 

# Local Environment Setup: JAR

`redis-server`
`java -jar sample-0.0.1-SNAPSHOT.jar --spring.profiles.active=local`

# Local Environment Setup: k3d[^1] [^2] [^3] [^4]

#### Build
```bash
brew install k3d
k3d version
alias k="kubectl"
```

```bash
# Description: Makefile for the springboot application

# Don't forget to export the environment variables secrets
# Don't forget to run redis

.PHONY: test build docker cluster secrets deploy

test:
	./gradlew test --info -Dspring.profiles.active=local

build:
	./gradlew build -Dspring.profiles.active=localk8

docker:
	docker build -t springboot --build-arg BASE_IMAGE="openjdk:17-jdk-slim" .

cluster:
	k3d cluster create local-cluster --port 8087:8087@loadbalancer

secrets:
	kubectl delete secret hcaptcha-secret --ignore-not-found
	kubectl create secret generic hcaptcha-secret --from-literal=HCAPTCHA_SECRET="$(HCAPTCHA_SECRET)"
	kubectl delete secret verify-service-sid --ignore-not-found
	kubectl create secret generic verify-service-sid --from-literal=VERIFY_SERVICE_SID="$(VERIFY_SERVICE_SID)"
	kubectl delete secret twilio-account-sid --ignore-not-found
	kubectl create secret generic twilio-account-sid --from-literal=TWILIO_ACCOUNT_SID="$(TWILIO_ACCOUNT_SID)"
	kubectl delete secret twilio-auth-token --ignore-not-found
	kubectl create secret generic twilio-auth-token --from-literal=TWILIO_AUTH_TOKEN="$(TWILIO_AUTH_TOKEN)"

secrets-check:
	kubectl get secrets
	kubectl get secret hcaptcha-secret -o yaml
	kubectl describe secret hcaptcha-secret
	#echo zz | base64 --decode

deploy:
	k3d image import springboot --cluster local-cluster
	helm install springboot ./springboot-chart -f ./springboot-chart/values-local.yaml

restart:
	kubectl rollout restart deployment springboot-app

log:
	kubectl describe pod -l app=springboot
	kubectl logs -l app=springboot --tail=100

uninstall:
	k3d cluster delete local-cluster

all: test build docker cluster secrets deploy
```

# Droplet Environment Setup: K3s

I upgraded to a droplet with 1 GB of RAM for $6 per month. And I've moved from systemd to K8s, Helm, and Docker. The Helm package is saved as a Github Release attachment. 

This line is needed else kubectl and k3s fails: `export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`[^5]

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
helm upgrade --install springboot springboot-chart-0.0.1.tgz
helm uninstall springboot
```

#### Container Registry
Docker image is saved to the Github Container Registry. Public container images on `ghcr.io` don't require authentication.[^6]

Also, Github container packages aren't deleted automatically, so will just pile up. Once a container image is over written, the original image becomes untagged from `latest`. The step `Delete untagged images` will then delete any untagged images. 

#### Cache
Github caching was added for gradle dependencies: [https://github.com/JacksonKuo/app-springboot/actions/caches](https://github.com/JacksonKuo/app-springboot/actions/caches)[^7]

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

#### Secrets
Secret management is handled via the following flow:
* Secrets stored in Github Secret
* Variable expansion in Github Action
* SSH'd into the droplet
* Create k8 secret via `kubectl create secret generic`
* Helm chart pulls k8 secret via `secretKeyRef` and create env var in container
* `application.properties`

#### Versioning
Right now I don't really want to deal with versioning. I'm just tagging everything as `latest`. As a consequence, the deployment spec sees that image tag hasn't changed and won't update the pod. Even though `imagePullPolicy` is set to `Always`, the setting only applies on pod creation. In order to update the pod, the deployment needs to be restarted: `kubectl rollout restart deployment spring-app`. 

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
* Redis
  * `redis-server --maxmemory 50mb`

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

[^4]: Note that k3d will add a `REDIS_PORT` environment var which will overwrite the `application.properties` file if `redis.port` field exists. Use `spring.redis.port` instead. Spring Boot will auto map `_` to `.`. 

[^5]: [https://stackoverflow.com/questions/76841889/kubectl-error-memcache-go265-couldn-t-get-current-server-api-group-list-get](https://stackoverflow.com/questions/76841889/kubectl-error-memcache-go265-couldn-t-get-current-server-api-group-list-get)

[^6]: [https://github.com/JacksonKuo/app-springboot/pkgs/container/springboot](https://github.com/JacksonKuo/app-springboot/pkgs/container/springboot)

[^7]: *GitHub will remove any cache entries that have not been accessed in over 7 days. There is no limit on the number of caches you can store, but the total size of all caches in a repository is limited to 10 GB:* [https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy)


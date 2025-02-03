---
layout: post
title: "Application Defense: Part IV - Deployment Pipeline 2.0"
date: 2025-1-22
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
* Step 4: **Upgrade deployment pipeline**

# Upgrade Deployment Pipeline

The next app defense upgrade will require running multiple services. The current pipeline with running services using shell scripts and systemd is fine for a simple app, but I'd like to gravitate toward docker and k8s. The problem with docker and especially k8s is the additional memory requirement. 

Currently I have 512 MB of RAM for $4 a month. Me being a penny pincher, I might upgrade to $6 for 1 GB of RAM, but I'm reluctant to spent much more. With only 1 GB of RAM, k8s will likely fail while trying to run other services. 

As for running on other platforms, services like AWS have more provenance, but have a much more complex cost structure and I don't want to run lots of infra in AWS for a long time due to fear of ballooning costs. Other platforms like fly.io just don't have enough learning upside, and is seldom seen in the industry. Using docker-compose.yml has a smaller footprint than k8s, but is also infrequently used in the industry versus just running kubernetes.

To address the RAM limitations, I'm be using K3s:
    * Local environment (macOS): k3d
    * Droplet environment: K3s

# Local Environment Setup: k3d[^1]

```bash
brew install k3d
k3d version
k3d cluster create local-cluster --port 8087:8097@loadbalancer[^2][^3]
k3d image import springboot --cluster local-cluster
docker exec k3d-local-cluster-server-0 crictl images
alias k="kubectl"
kubectl apply -f deployment.yaml

kubectl delete -f deployment.yaml
k3d cluster delete local-cluster
```

Minimal K8 manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-app
spec:
  selector:
    matchLabels:
      app: springboot
  template:
    metadata:
      labels:
        app: springboot
    spec:
      containers:
        - name: springboot-container
          image: springboot
          imagePullPolicy: Never
          ports:
            - containerPort: 8087
---
apiVersion: v1
kind: Service
metadata:
  name: springboot-service
spec:
  type: LoadBalancer
  selector:
    app: springboot
  ports:
    - port: 8087
      targetPort: 8087
```

# Droplet Environment Setup: K3s




# References

[^1]: [https://www.perdian.de/blog/2023/07/06/running-kubernetes-locally-on-a-mac-with-k3d/](https://www.perdian.de/blog/2023/07/06/running-kubernetes-locally-on-a-mac-with-k3d/)

[^2]: The `~/.kube/config` created points to `host.docker.internal` which does not resolve correctly. Short fix is to change the address to `localhost`. Long-term fix is to edit `/etc/host` with `127.0.0.1 host.docker.internal`

[^3]: Alternative to using loadbalancer is port-forwarding: `kubectl port-forward svc/springboot-service 8087:8087`

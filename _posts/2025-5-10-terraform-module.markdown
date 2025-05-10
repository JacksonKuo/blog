---
layout: post
title: "Terraform: Multi-Cloud Module"
date: 2025-5-10
tags: ["IaC"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Create a terraform module for `app-springboot` deployment that allows me to swap cloud environments.

# Interface
My expected interface is below, effectively one main variable `cloud_provider` which will dictate which cloud platform the infrastructure will be created in. Note that when different cloud environments are introduced the CI/CD pipeline will need to be modified. Currently the pipeline works with a droplet

```yaml
module springboot_app {
    source = "../../modules/multicloud/cluster"
    providers = {
        digitalocean = digitalocean
        github = github
    }
    cloud_provider = "digitalocean"
}
```

# Design
According to Terraform: Up & Running 3rd edition, multi-cloud modules are a bit frowned upon. 



# References
[^1]: [][]
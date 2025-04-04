---
layout: post
title: "Terraform Infrastructure: Part I - Droplet"
date: 2025-4-3
tags: ["app"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Build a bunch of infrastructure as code (IaC) using Terraform. 

* Step 1: **Terraform Droplet**

# Cloud Deployment Pipeline
Let's add Terraform to do the initial Digital Oceans infrastructure setup with the option to be able to swap between different cloud ecosystems. This probably would make more sense with Pulumi, but I kinda want to write a Terraform module for this.

To start out with, I'm just saving and running the Terraform locally, no workflows, S3/Dynamo, Atlantis, or Spacelift.

1. DO PAT Token for Droplet
2. Github PAT Token for Secret
3. Terraform
    * Create droplet 1GB
    * Update A Records
    * Get SSH key
    * Get IP address
    * Upload to Github secrets
    * Restart app pipeline

# DO PAT Token
`terraform-do-droplet-pat` token scope:
* actions: read
* domain: create, read, update, delete
* droplet: create, read, update, delete
* regions: read
* sizes: read
* ssh_key: read

# Github PAT Token 
`terraform-gh-springboot-pat` fine-grained token scope:
* meta: read
* secret: read, write

# Terraform
[https://github.com/JacksonKuo/terraform](https://github.com/JacksonKuo/terraform)

#### Droplet
```
resource "digitalocean_droplet" "droplet" {
  image   = "debian-12-x64"
  name    = "debian-s-1vcpu-1gb-nyc1-01"
  region  = "nyc1"
  size    = "s-1vcpu-1gb"
}
```
Repository Secrets
DROPLET_IP
DROPLET_SSH_PRIVATE_KEY

# References
[^1]: [https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/droplet](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/droplet)
[^1]: [https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/data-sources/droplet](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/data-sources/droplet)
[^1]: [https://registry.terraform.io/providers/integrations/github/latest/docs](https://registry.terraform.io/providers/integrations/github/latest/docs)
[^1]: []()
[^1]: []()
[^1]: []()


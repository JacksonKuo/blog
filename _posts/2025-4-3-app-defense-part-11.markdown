---
layout: post
title: "Terraform: DigitalOcean Droplet"
date: 2025-4-3
tags: ["IaC"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's add Terraform to do the initial Digital Oceans infrastructure setup. This will allow me to tear down and up the droplet without manual action.[^1] Terraform is run locally. No workflows, S3/Dynamo, Atlantis, or Spacelift. Note all PAT tokens are kept locally. 

[https://github.com/JacksonKuo/terraform](https://github.com/JacksonKuo/terraform)

#### Create DigitalOcean PAT token 
`terraform-do-droplet-pat` token scope:
```
* actions: read
* domain: create, read, update, delete
* droplet: create, read, update, delete
* regions: read
* sizes: read
* ssh_key: read
```

#### Create Github PAT token 
`terraform-gh-springboot-pat` fine-grained token scope:
```
* meta: read
* secret: read, write
```

# Terraform
```bash
export TF_VAR_do_token=
export TF_VAR_gh_token=

terraform init
terraform init -upgrade
terraform plan
terraform apply
terraform destroy
```

#### Create droplet[^2] [^3]
```bash
resource "digitalocean_droplet" "droplet" {
  image  = "debian-12-x64"
  name   = "debian-s-1vcpu-1gb-nyc1-01"
  region = "nyc1"
  size   = "s-1vcpu-1gb"
  ssh_keys = [
    data.digitalocean_ssh_key.droplet.id
  ]
}
```
Pull public `ssh_key` uploaded to DO.

#### Update A records
```
A bakacore.com 3600
A www.bakacore.com 3600
```

#### Upload Droplet IP and SSH key to Github secrets
* `DROPLET_IP`
* `DROPLET_SSH_PRIVATE_KEY`

Droplet specific private key is loaded locally
```bash
resource "github_actions_secret" "droplet_ip" {
  repository       = "app-springboot"
  secret_name      = "DROPLET_IP"
  plaintext_value  = digitalocean_droplet.droplet.ipv4_address
}

locals {
  droplet_ssh_private_key = file("/Users/jacksonkuo/.ssh/id_ed25519_droplet")
}

resource "github_actions_secret" "droplet_ssh_private_key" {
  repository       = "app-springboot"
  secret_name      = "DROPLET_SSH_PRIVATE_KEY"
  plaintext_value  = local.droplet_ssh_private_key
}
```

#### Automate certbot

```bash
  echo 'Check certbot...'
  if test -d /etc/letsencrypt/live/bakacore.com; then
    echo 'Certbot already run...'
  else
    echo 'Running certbot...'
    apt-get -y install certbot
    certbot certonly --standalone -d bakacore.com --agree-tos --register-unsafely-without-email
    sudo openssl pkcs12 -export \
      -in /etc/letsencrypt/live/bakacore.com/fullchain.pem \
      -inkey /etc/letsencrypt/live/bakacore.com/privkey.pem \
      -out /etc/letsencrypt/live/bakacore.com/keystore.p12 \
      -name springboot \
      -password pass:<placeholder>
  fi
```

# References
[^1]: [https://registry.terraform.io/providers/integrations/github/latest/docs](https://registry.terraform.io/providers/integrations/github/latest/docs)

[^2]: [https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/droplet](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/droplet)

[^3]: [https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/data-sources/droplet](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/data-sources/droplet)





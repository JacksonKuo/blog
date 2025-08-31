---
layout: post
title: "Ephemeral GitHub Tokens: Vault Plugin"
date: 2025-9-2
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
A walkthrough of the vault plugin github secret: [https://github.com/martinbaillie/vault-plugin-secrets-github](https://github.com/martinbaillie/vault-plugin-secrets-github)

# Setup
[https://developer.hashicorp.com/vault/tutorials/get-started/setup](https://developer.hashicorp.com/vault/tutorials/get-started/setup)

`ssh -i ~/.ssh/id_ed25519 root@<droplet>`

[https://developer.hashicorp.com/vault/install](https://developer.hashicorp.com/vault/install)

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault`
```

```bash
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 \
    -nodes -keyout /tmp/vault-key.pem -out /tmp/vault-cert.pem \
    -subj "/CN=localhost" \
    -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

```json
cat > /tmp/vault-server.hcl << EOF
api_addr                = "https://<droplet>:8200"
cluster_addr            = "https://127.0.0.1:8201"
cluster_name            = "learn-vault-cluster"
disable_mlock           = true
ui                      = true

listener "tcp" {
address       = "0.0.0.0:8200"
tls_cert_file = "/tmp/vault-cert.pem"
tls_key_file  = "/tmp/vault-key.pem"
}

backend "raft" {
path    = "/tmp/vault-data"
node_id = "learn-vault-server"
}
EOF
```

```bash
vault server-config=/tmp/vault-server.hcl

export VAULT_ADDR=https://127.0.0.1:8200
export VAULT_SKIP_VERIFY=true
vault operator init -key-shares=1 -key-threshold=1
vault operator unseal
vault status
vault login
```
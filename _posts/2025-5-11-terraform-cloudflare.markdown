---
layout: post
title: "Terraform: Cloudflare Migration"
date: 2025-5-10
tags: ["IaC"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Migrate bakacore.com to Cloudflare.

#### Current Setup

* GoDaddy
    - NS,bakacore.com, ns1.digitalocean.com
* DigitalOcean
    - A Record, bakacore.com, IPv4
    - A Record, www.bakacore.com, IPv4
    - NS, bakacore.com, ns1.digitalocean.com. (autocreated by DO)

#### Design

DNS Setups[^3]

DNS only
    MX and NS
Proxied 
    A Records, bakacore.com
    A Records, www.bakacore.com

Registar
    destiny.ns.cloudflare.com
    leo.ns.cloudflare.com

Registrars take up to 24 hours to process nameserver changes

Encryption mode[^2]

Current encryption mode:

Full


The Recommender crawls your site using the Cloudflare-SSLDetector user agent, recognized as a trusted bot by Cloudflare, and bypasses robots.txt rules (except those that specifically target it) to ensure accuracy.

#### Pricing

Cloudflare Free Plan[^1], $0/month, Basic protection and performance for personal or simple websites.


#### TLS?
Certbot + Cloudflare

# Terraform - Cloudflare



# References
[^1]: [https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/)

[^2]: [https://www.cloudflare.com/plans/#compare-features](https://www.cloudflare.com/plans/#compare-features)


[^1]: [https://developers.cloudflare.com/dns/zone-setups/](https://developers.cloudflare.com/dns/zone-setups/)
[^1]: [https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/)
[^1]: []()
[^1]: []()
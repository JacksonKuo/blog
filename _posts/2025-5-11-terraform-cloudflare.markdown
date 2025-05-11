---
layout: post
title: "Terraform: Cloudflare Zone"
date: 2025-5-11
tags: ["IaC"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Configure `bakacore.com` to use Cloudflare.

# Configuration

#### DNS Setup
The old setup uses a GoDaddy registrar pointing to DO nameservers: `NS,bakacore.com, ns1.digitalocean.com`. DO DNS has A Records and another set of NS records that were generated automatically by DO. 

* GoDaddy
    - `NS,bakacore.com, ns1.digitalocean.com`
* DigitalOcean
    - `A Record, bakacore.com, IPv4`
    - `A Record, www.bakacore.com, IPv4`
    - `NS, bakacore.com, ns1.digitalocean.com.`

The new setup is a Full DNS Setup that uses CF as the authoritative DNS nameservers.[^1] [^2] When the DNS setup is Proxied, TTL is always set to Auto. 

* GoDaddy
    - `NS, bakacore.com, destiny.ns.cloudflare.com.`
    - `NS, bakacore.com, leo.ns.cloudflare.com.`
* Cloudflare
    - `A Record, bakacore.com, Status: Proxied, TTL: Auto`
    - `A Record, www.bakacore.com, Status: Proxied, TTL: Auto`

#### Pricing
Pricing is pretty nice: Free Plan - $0/month *(basic protection and performance for personal or simple websites)*[^3]

#### Encryption
`Browser` <*connection #1*> `Cloudflare` <*connection #2*> `Origin`

The encryption mode is set to `Full (Strict)`[^4] [^5]. 
* `Full`: Allows HTTP/HTTPS, does not validate the Origin HTTPS cert
* `Full (Strict)`: Allows HTTP/HTTPS, validates the Origin HTTPS cert
* `Strict (SSL-Only Origin Pull)`: Only HTTPS, validates the Origin HTTPS cert

Since we already have a valid public certificate from Certbot, `Full (Strict)` makes sense, since I might want to set a HTTP endpoint in the future. 

I also noticed that Connection #1 used a Google certificate. Turns out that Cloudflare's Universal Certificate can use Google Trust Services.[^7]

#### Terraform
CF DNS records require a `zone_id`, which can be pulled using a `filter` attribute.[^8] The `zone_id` isn't neccesarry sensitive, more system information, but I'll still pull the value using terraform instead of hard-coding in source. 

```yaml
resource "cloudflare_dns_record" "bakacore" {
  zone_id = data.cloudflare_zone.bakacore.zone_id
  content = var.droplet_ip
  name = "@"
  proxied = true
  ttl = 1
  type = "A"
}

data "cloudflare_zone" "bakacore" {
  filter = {
    name = "bakacore.com"
  }
}
```

#### Issues
I ran into two minor issues. The first being that Cloudflare only supports proxying HTTPS using the following ports: 443, 2053, 2083, 2087, 2096, 8443.[^6] I was originally on `8087`, and had to change over to `8443`. 

However going to `https://bakacore.com:443` shows this unappealing Cloudflare *Web server is down* page. There is apparently is no way to turn this page off without using a Cloudflare worker. Oh well, it's nothing I really care about that much right now..

The second issue was that my local laptop DNS was using a DNS server set by my routers ISP. This IPv6 nameserver was frequently offline for a few minutes and back online for 5 mins and would continuous cycle between down and available. This unstableness resulted in a random `DNS_PROBE_FINISHED_NXDOMAIN` errors when trying to resolve `bakacore.com`. I wwitched my default DNS server to `1.1.1.1` which fixed the issue.

# References
[^1]: [https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/)

[^2]: [https://developers.cloudflare.com/dns/zone-setups/](https://developers.cloudflare.com/dns/zone-setups/)

[^3]: [https://www.cloudflare.com/plans/#compare-features](https://www.cloudflare.com/plans/#compare-features)

[^4]: [https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/)

[^5]: [https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/)

[^6]: [https://developers.cloudflare.com/fundamentals/reference/network-ports/](https://developers.cloudflare.com/fundamentals/reference/network-ports/)

[^7]: [https://developers.cloudflare.com/ssl/reference/certificate-authorities/](https://developers.cloudflare.com/ssl/reference/certificate-authorities/)

[^8]: [https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs/data-sources/zone](https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs/data-sources/zone)
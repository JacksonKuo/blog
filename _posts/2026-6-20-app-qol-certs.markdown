---
layout: post
title: "Application QoL: Auto-Renew Certificates"
date: 2026-6-20
tags: ["app"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Improve the quality of life of my app environment [https://bakacore.com:8443/](https://bakacore.com:8443/) by adding cert auto-renewal.

# Breakdown
Two places where we really need this:
* `certbot`
* `k3s`

# Certbot
What's happening is `certbot` will auto-update the PEM file, but my springboot app is configured to accept the cert via `.p12` file.

The original cert deployment:
```shell
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
    -password pass:foobar
fi
```

The new cert deployment doesn't use `.p12` anymore:
```shell
echo 'Ensure certbot is installed and cert exists...'
apt-get -y install certbot
if ! test -d /etc/letsencrypt/live/bakacore.com; then
echo 'Running certbot...'
certbot certonly --standalone -d bakacore.com --agree-tos --register-unsafely-without-email
fi
# renew if due (no-op until ~30 days before expiry), then make the
# private key readable by the non-root chainguard user in the pod
certbot renew --quiet
# let the non-root pod user traverse into live/ and archive/ (certbot
# locks these to 0700) and read the private key
chmod 755 /etc/letsencrypt/live 
chmod 755 /etc/letsencrypt/archive
chmod 644 /etc/letsencrypt/live/bakacore.com/privkey.pem
```

SpringBoot does support PEM, and the [application.properties](https://github.com/JacksonKuo/app-springboot/blob/main/src/main/resources/application-prod.properties) has been updated with:
* `spring.ssl.bundle.pem.letsencrypt.keystore.certificate`
    - `/etc/letsencrypt/live/bakacore.com/fullchain.pem`
* `spring.ssl.bundle.pem.letsencrypt.keystore.private-key`
    - `/etc/letsencrypt/live/bakacore.com/privkey.pem`

Also the [volumeMount](https://github.com/JacksonKuo/app-springboot/blob/main/springboot-chart/templates/deployment-app.yaml) has been updated to include the `/archive` folder. Which apparently the `/live` directory symlinks to. Certbot, by default, has a webhook that should auto-renew the PEMs. The PEMs are accessible via volumeMount and the file and dir permissions should still carry over to the next refresh. 

# K3s
I didn't know this but K3s certs expire every 3 months and there is no auto-renew. 

Let's add a cronjob

```shell
echo 'Install weekly K3s cert-rotation cron (renews between deploys)...'
echo '0 4 * * 0 root openssl x509 -checkend 2592000 -noout -in /var/lib/rancher/k3s/server/tls/client-admin.crt || systemctl restart k3s' > /etc/cron.d/k3s-cert-rotate
```

Every Sunday at 4am, as `root` run `openssl x509` with option `-checkend` which will see if cert is expired in 30 days (2592000 secs) and if true, restart k3s service. 
---
layout: post
title: "Application Defense: Part II - Deployment Pipeline 1.0"
date: 2025-1-21
tags: ["app"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement

Build a bunch of application defensive mechanisms. This problem can be further broken down into:

* Step 1: Build a sample app with Spring Boot
* Step 2: **Build a deployment pipeline**

# Deployment Pipeline

My deployment pipeline will use Github Actions. The goal is to make a quick simple deployment pipeline so that CI/CD is available for my Spring Boot app, in order to start building and testing some application defenses.

* [https://github.com/JacksonKuo/app-springboot/tree/v1.0.0](https://github.com/JacksonKuo/app-springboot/tree/v1.0.0)
* [https://github.com/JacksonKuo/app-springboot/blob/main/.github/workflows/gradle.yml](https://github.com/JacksonKuo/app-springboot/blob/main/.github/workflows/gradle.yml)

```yaml
#.github/workflows/gradle.yml
name: Build

on: [ push ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout src
      uses: actions/checkout@v4

    - name: Set up JDK 
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Build with Gradle
      run: |
        chmod +x ./gradlew
        ./gradlew build

    - name: Publish to Latest Release
      run: |
        if gh release view latest; then
          echo "Release exists. Deleting old release..."
          gh release delete latest --yes
        fi

        gh release create latest build/libs/sample-0.0.1-SNAPSHOT.jar \
          --title "Latest Release" \
          --notes "This is the latest build of the application." \
          --prerelease
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    - name: SSH
      run: |
        echo 'Setup SSH...'; 
        install -m 600 -D /dev/null ~/.ssh/id_ed25519
        echo "$DROPLET_SSH_PRIVATE_KEY" > ~/.ssh/id_ed25519
        ssh -o StrictHostKeyChecking=no root@$DROPLET_IP "
        
        echo 'Install JDK...'; 
        apt -qq update && apt-get -qq install -y openjdk-17-jdk-headless;
        echo 'Downloading the latest JAR...'; 
        
        echo 'Downloading the latest JAR...';
        curl -L -o /root/sample-0.0.1-SNAPSHOT.jar \
        https://github.com/JacksonKuo/app-springboot/releases/download/latest/sample-0.0.1-SNAPSHOT.jar &&
        
        echo 'Restarting the web application...'; 
        systemctl stop webapp || true;
        systemctl reset-failed webapp || true;
        systemd-run --unit=webapp --description='Transient Web App Service' --setenv=HCAPTCHA_SECRET=$HCAPTCHA_SECRET --property=Restart=always java -jar /root/sample-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod"
      env:
        DROPLET_IP: ${{ secrets.DROPLET_IP }}
        DROPLET_SSH_PRIVATE_KEY: ${{ secrets.DROPLET_SSH_PRIVATE_KEY }}
        HCAPTCHA_SECRET: ${{ secrets.HCAPTCHA_SECRET }}
```

#### Artifacts

The workflow will generate a JAR file that will be saved in the Github Releases as a publicly available attachment.[^1] The workflow does need read / write permissions in order to create Releases. 

* Github Artifacts and Packages don't have a mechanism to download without authentication. I want a internet facing artifact that can be downloaded without any sort of Github Token. The idea is I should be able to download and run this JAR from any platform anywhere at any time.
* Note to self, when calling the Github API always use Github CLI instead of direct `curl` calls. The code simplicity with using `gh cli` is like night and day.

#### Hosting

The hosting platform will be the least expensive Digital Ocean droplet / 512 MB Memory / $4 per month. 

* I'm choosing a droplet over App Platform because I intend to add other local services in the future
* The droplet has 512 MB Memory, which means I can't directly build the JAR on the droplet. The gradle daemon seems to run into errors with such low memory available. From testing, 1GB RAM minimum, 2GB to be able to build with tests.

#### Deployment

Github Action will SSH into the droplet in order to download and run the JAR. 

* The SSH keypair was created specifically for the droplet, so no risk serious risk if the private key is compromised.
    * `ssh-keygen -t ed25519 -C "droplet"`
* I'm using `systemd-run`, so I don't have to manually create a systemd unit file
* The web service is stopped, the failed state is cleared, new JAR downloaded, and service restarted using the prod properties `--spring.profiles.active=prod`

#### TLS Encryption

I also setup Let's Encrypt using certbot.

```
apt update
apt-get -y install certbot
certbot certonly --standalone -d bakacore.com --agree-tos --register-unsafely-without-email
sudo openssl pkcs12 -export \
    -in /etc/letsencrypt/live/bakacore.com/fullchain.pem \
    -inkey /etc/letsencrypt/live/bakacore.com/privkey.pem \
    -out /etc/letsencrypt/live/bakacore.com/keystore.p12 \
    -name springboot \
    -password pass:foobar
```

Integrating with Spring Boot requires the following settings

```properties
#src/main/resources/application-prod.properties
server.ssl.enabled=true
server.ssl.key-store=/etc/letsencrypt/live/bakacore.com/keystore.p12
server.ssl.key-store-password=foobar
server.ssl.key-store-type=PKCS12
```

Certbot also installs a script in `/etc/cron.d/` that will auto-renew the certificate before the 90 day expiration[^1].

#### Secret Management

Secrets are stored in Github repository secrets. The `DROPLET_*` values are manually set when the droplet is created:

* DROPLET_IP
* DROPLET_SSH_PRIVATE_KEY

# Docker

I have a Dockerfile that can be used to build the app locally. The gradle build cache isn't setup to work with Docker so all builds take around 30 seconds. 

```Dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app
COPY . /app
RUN chmod +x gradlew
RUN ./gradlew build --info --console=plain -Dorg.gradle.jvmargs="-Xmx512m -XX:MaxMetaspaceSize=512m" 
EXPOSE 8443
CMD ["java", "-jar", "build/libs/sample-0.0.1-SNAPSHOT.jar", "--spring.profiles.active=local"]
```

```bash
apt-get install git
apt-get install docker.io
docker build -t springboot .
docker run -p 8443:8443 springboot
```

# Future Improvements

* I'm not using Dockerfiles in the pipeline to build an image or to run the image for simplicity
* I'm not using Terraform to create or setup the droplet
* I'm not using a full-fledge artifact manager like Artifactory

# References

[^1]: [https://www.digitalocean.com/community/tutorials/how-to-use-certbot-standalone-mode-to-retrieve-let-s-encrypt-ssl-certificates-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-use-certbot-standalone-mode-to-retrieve-let-s-encrypt-ssl-certificates-on-ubuntu-20-04)

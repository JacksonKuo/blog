---
layout: post
title: "Application Defense: Part III - hCaptcha"
date: 2025-1-22
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
* Step 3: **Add hCaptcha**

# hCaptcha

Integation hCaptcha on a Spring Boot web endpoint:

* hCaptcha documentation: 
    * [https://docs.hcaptcha.com/](https://docs.hcaptcha.com/)
* hCaptcha endpoint: 
    * [https://bakacore.com:8443/captcha](https://bakacore.com:8443/captcha)
* CaptchaController: 
    * [https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/CaptchaController.java](https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/CaptchaController.java)

The free mode of hCaptcha only allows "Always Challenge" mode. 

The client-side JS requires a sitekey. The server-side backend communicates with `/siteverify` and requires the sitekey secret and client response. 

```bash
curl https://api.hcaptcha.com/siteverify \
  -X POST \
  -d "secret=YOUR-SECRET&remoteip=CLIENT-IP&response=CLIENT-RESPONSE"
```

# Secret Management

The Spring Boot app requires the hCaptcha sitekey secret. This secret is safely passed via environment variables to the `application.properties` config file and then to the `hcaptchaSecret` field via the @Value annotation[^1]:

```java
    @Value("${hcaptcha.secret}")
    private String hcaptchaSecret;
```

* In production, the environment variable is initially sourced from Github repo secrets. Note that `systemd-run` creates a new process for the webapp, so in order to load the secret into the app, the secret must be passed to `systemd-run` via the `setenv` argument
* In the local Dockerfile build, the secret is passed via `docker run -e`. 
    * In MacOS, to load the hCaptcha from localhost edit the following file:[^2]
    ```properties
    #/private/etc/hosts
    127.0.0.1       local.bakacore.com
    ```

* In order to pass build tests the `application.properties` file has a default value set `hcaptcha.secret=${HCAPTCHA_SECRET:fill-me}`

```bash
java -jar myapp.jar --spring.profiles.active=local
docker run -e HCAPTCHA_SECRET=$HCAPTCHA_SECRET -p 8443:8443 springboot
```

# Future Improvements

* Add tests for the captcha using test keys: [https://docs.hcaptcha.com/#integration-testing-test-keys](https://docs.hcaptcha.com/#integration-testing-test-keys)
* Test bypasses with: [https://github.com/xrip/playwright-hcaptcha-solver](https://github.com/xrip/playwright-hcaptcha-solver)

# References

[^1]: *"One of the most common uses for @Value is injecting values from properties files."* [https://medium.com/@AlexanderObregon/how-to-use-the-value-annotation-in-spring-d5547c7b5e4b](https://medium.com/@AlexanderObregon/how-to-use-the-value-annotation-in-spring-d5547c7b5e4b)

[^2]: [https://docs.hcaptcha.com/#local-development](https://docs.hcaptcha.com/#local-development)

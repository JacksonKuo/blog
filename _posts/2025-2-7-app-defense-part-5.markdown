---
layout: post
title: "Application Defense: Part V - Custom Rate Limit"
date: 2025-2-7
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
* Step 3: Add hCaptcha
* Step 4: Upgrade deployment pipeline with K8s
* Step 5: **Add custom rate limit**

# Custom Rate Limit using Redis
Build a custom rate limit for a Spring Boot endpoint with a focus on simplicity, efficiency, and speed. 

These types of fine-grained application rate limits are often needed because WAFs can only block request above a certain rate. For example AWS WAF, used to only rate limit at a minimal of 100 requests per window (1/2/5/10 minutes).[^1] Now AWS WAF can rate limit on 10 requests per window, but even that for certain situations is still not granular enough.[^2] 

There's a number of different options for implementation rate limits. Options like Bucket4J and Resilience4J are primarily run in-memory and require additional code to integrate with redis. And while in-memory is faster, I want the rate limit to be distributed in the future potentially. I settled on using Redis + Redisson.

There's multiple ways to use Redisson to build a rate limit

* RateLimiter (builtin)
* RAtomicLong
* Lua scripts

For simplicity, let's see how capable RateLimiter is: [https://redisson.org/docs/data-and-services/objects/#ratelimiter](https://redisson.org/docs/data-and-services/objects/#ratelimiter)

# Redis + Redisson + RateLimiter
I created a RateLimiter service that accepts 3 requests per 1 minute for each IP address. The `RateType` is set to `OVERALL` instead of `PER_CLIENT` however really the `RateType` doesn't actually matter, instead using one global key or multiple client keys is what's actually important. The service calls a `RedissonClient` bean in the `RedisConfig`. 

```java
    public RRateLimiter getRateLimiter(String clientIp) {
        String key = "rate-limiter:" + clientIp; 
        RRateLimiter rateLimiter = redissonClient.getRateLimiter(key);
        rateLimiter.trySetRate(RateType.OVERALL, 3, 1, RateIntervalUnit.MINUTES);
        redissonClient.getBucket(key).expire(2, TimeUnit.MINUTES); 
        return rateLimiter;
    }
```

Due to memory constraints, I'm not running Traefik as my load balancer. K3s apparently will then default to Klipper LB.[^3] When retrieving IP addresses what will happen is the internal LB IP address is returned: `String clientIp = request.getRemoteAddr();` Normally, the frontend like Cloudfront will have some `True-Client-IP` header that can be referenced. In our case, we'll need to set `externalTrafficPolicy: Local` on the service load balancer in order to preserve the original IP address.[^4] The key is set to expire after 3 minute, which as the same rate limit window, in order to not fill up redis with client IP keys. 

# Thoughts
The official documentation and examples for RateLimiter seems to be a little lacking. The ratelimiter doc doesn't even have an example for `PER_CLIENT`, and as stated before, the `RateType` doesn't actually matter, instead using one global key or multiple client keys is what's actually important.[^5] And having to set the key expiration and not being able to set the TTL during key creation is a bit annoying. Actually it creates a race condition that I'll need to fix if the expiration is at 1 minute. 

If I needed more control over the rate limit, I think I would just use `RAtomicLong` and the Java versions of `INCR` and `TTL`. 

# References
[^1]: [https://aws.amazon.com/about-aws/whats-new/2024/08/aws-waf-rate-based-rules-lower-rate-limits/](https://aws.amazon.com/about-aws/whats-new/2024/08/aws-waf-rate-based-rules-lower-rate-limits/)

[^2]: [https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based-high-level-settings.html](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based-high-level-settings.html)

[^3]: [https://github.com/k3s-io/klipper-lb](https://github.com/k3s-io/klipper-lb)

[^4]: [https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/)

[^5]: [https://redisson.org/docs/data-and-services/objects/#ratelimiter](https://redisson.org/docs/data-and-services/objects/#ratelimiter)

---
layout: post
title: "Application Defense: Part VIII - Test Suite"
date: 2025-2-14
tags: ["app"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Build a bunch of application defensive mechanisms. This problem can be further broken down into:

* Step 1: Build a sample app with Spring Boot
* Step 2: Build a deployment pipeline
* Step 3: Add hCaptcha
* Step 4: Upgrade deployment pipeline
* Step 5: Add custom rate limit
* Step 6: Add Twilio MFA
* Step 7: Add Smokescreen
* Step 8: Add test suite

# Test Suite
As typical of Spring Boot apps, I'l be using JUnit and Mockito. There are 4 services that could use tests:

* Rate limit
* Smokescreen
* Twilio
* hcaptcha

# Test Methodology
To help me on thinking about and writing tests, i read through most of [Unit Testing Principles, Practices, and Patterns](https://www.amazon.com/gp/product/B09782L692/) by Vladimir Khorikov, which was very enlightening. That said, a lot of the books guidance hasn't fully sunk in, so I'm sure my tests and code will still have all sorts of problems. Regardless, let's get to it!

Oh lastly, for Spring Boot testing here are a bunch of references I used:

* [https://www.baeldung.com/spring-boot-testing](https://www.baeldung.com/spring-boot-testing)
* [https://medium.com/twodigits/springboottest-is-a-code-smell-cf0498766a6c](https://medium.com/twodigits/springboottest-is-a-code-smell-cf0498766a6c)
* [https://reflectoring.io/spring-boot-test/](https://reflectoring.io/spring-boot-test/)[^1]

#### Rate limit
The first test is the redis rate limit. There's an autowired `RateLimitService` that accepts a `RedissonClient` in the constructor, and has a `getRateLimiter()` method. A `RedisConfig` that creates a `RedissonClient` bean. And a `RateLimitController` then accepts `RateLimitService` in the constructor, and that has a `/ratelimit` endpoint. 

This service doesn't have a lot of complex business logic, imo. However depending on the app, the rate limit could be a core component.

Only writing unit tests would be great, however since we have the redis dependency I'll either have to mock it or run integration tests. The main logic is centered around redis calls, with little to no complex domain logic that warrant a unit test. Additionally security tests often lean towards integration tests anyway. 

The main functionality I want to test is:
* Successful permit for request 1-3
* Fail permit for request 4
* Check if TTL is set

#### Smokescreen

#### Twilio

#### hcaptcha

# References
[^1]: *(As of Spring Boot 2.1, we no longer need to load the SpringExtension because it's included as a meta annotation in the Spring Boot test annotations like @DataJpaTest, @WebMvcTest, and @SpringBootTest.)*
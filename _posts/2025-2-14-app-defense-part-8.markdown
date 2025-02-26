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
* [https://reflectoring.io/spring-boot-web-controller-test/](https://reflectoring.io/spring-boot-web-controller-test/)
* [https://spring.io/guides/gs/testing-web](https://spring.io/guides/gs/testing-web)

Also I added a `.vscode/tasks.json` to help build and test within the IDE:
* [https://github.com/JacksonKuo/app-springboot/blob/main/.vscode/tasks.json](https://github.com/JacksonKuo/app-springboot/blob/main/.vscode/tasks.json)

# Rate limit
The first test is the redis rate limit. There's an autowired `RateLimitService` that accepts a `RedissonClient` in the constructor, and has a `getRateLimiter()` method. A `RedisConfig` that creates a `RedissonClient` bean. And a `RateLimitController` then accepts `RateLimitService` in the constructor, and that has a `/ratelimit` endpoint. 

This service doesn't have a lot of complex business logic, imo. However depending on the app, the rate limit could be a core component.

Only writing unit tests would be great, however since we have the redis dependency I'll either have to mock it or run integration tests. The main logic is centered around the redis operators, with little to no complex domain logic that warrant a unit test. Additionally security tests often lean towards integration tests anyway. 

The main functionality I want to test is:
* Successful permit for request 1-3
* Fail permit for request 4
* Check if TTL is set

#### TestContainers
My first iteration used `@BeforeAll` and `@AfterAll` to start and stop `redis-server` locally.[^2] However, dealing with local fixtures is super annoying. So i've shifted over to [Testcontainers](https://testcontainers.com/). The setup was a lot easier than I thought it would be.

```java
public static final DockerImageName REDIS_IMAGE = DockerImageName.parse("redis:latest");

static GenericContainer<?> redis = new GenericContainer<>(REDIS_IMAGE).withExposedPorts(6379);
```

#### Error handling
Since my RedissionClient bean originally requires a running `redis-server` any time the server wasn't available the bean would fail, which would cause tests to fail and any @SpringBootTests to fail. I added additional error handling, specifically

* If `RedissionClient` throws an Exception then return `null` 
* `@Autowired(required = false)`[^3]

```java
@Autowired(required = false)
public RateLimitService(@Autowired(required = false) RedissonClient redissonClient) {
    this.redissonClient = redissonClient;
}
```

Even though Testcontainer should alleviate any issues now, it's nice to have check to prevent all `@SpringBootTest` failing. Though i'm not sure if `required = false` is the best way to handle this issue. 

#### Performance

Instead of call `@SpringBootTest` which will load the entire application context, I use `@ContextConfiguration` to manually load my beans. Unlike `@SpringBootApplication`, `@ContextConfiguration` doesn't automatically load properties files, which is why `@TestPropertySource` points to `application-local.properties`.

```java
@SpringJUnitConfig
@ContextConfiguration(classes = { RedisConfig.class, RateLimitService.class })
@ActiveProfiles("local")
@TestPropertySource("classpath:application-local.properties")
public class RateLimitServiceIT {
    ...
    // arrange
    String clientIp = "1.1.1.1";
    // act
    boolean permitSuccess = rateLimitService.getRateLimiter(clientIp).tryAcquire(3);
    boolean permitFail = rateLimitService.getRateLimiter(clientIp).tryAcquire(1);
    // assert
    assertTrue(permitSuccess);
    assertFalse(permitFail);
```

A second test checks if a TTL exists. 

``` java
    long ttl = rateLimitService.getTimeToLive(clientIp);
    assertTrue(ttl > 0, "TTL should be set and greater than zero");
```

In the future I'll use JUnit `@ParameterizedTest`, but for now I just want to go the old fashion way.

# Smokescreen
Alright, next is up is smokescreen. Smokescreen isn't a service class like redis, but instead is a proxy class where `WebProxy.class` takes in a `WebClient.class` that is created in `WebClientConfig.java`. I follow the same pattern of using Testcontainers since there's an image on `ghcr.io`. 

```java
public static final DockerImageName SMOKESCREEN_IMAGE = DockerImageName.parse("ghcr.io/jacksonkuo/smokescreen:latest");

static GenericContainer<?> smokescreen = new GenericContainer<>(SMOKESCREEN_IMAGE).withExposedPorts(4750);
```

A couple of miscellaneous things I learned:
* `@MockBean` is considered deprecated. Spring docs recommend using `@MockitoBean`[^4]
* When setting the `spring.smokescreen.host` and `spring.smokescreen.port=4750` values in `application.properties` the prefix `spring.` is required, else a weird error occurs[^5]

The integration tests don't have to deal with mocks and run relatively fast. If my smokescreen image has issues, the integration tests will alert which is kinda nice.

```java
    @Test
    public void testSmokescreenAllow() {
        System.err.println(smokescreen.getHost() + smokescreen.getMappedPort(4750));
        Mono<String> responseMono = webProxy.retrieveUrl("https://example.com");
        String response = responseMono.block();
        assertThat(response.startsWith("<!doctype html>")).isTrue();
    }
```

I also marked Tag the class with `@Tag("integration")`, in case I ever want to run only integration tests. 

# Twilio

I don't really want to do E2E tests for this. For one reason, I'm using a Twilio trial account, and I only have $15.2346 left. Though Twilio does have test credentials that don't charge[^6]

#### Logging / Debugging

I move the Twilio code from the controller to it's own service. I ran into a few issues while triaging so I put in some slf4j logging. 

`private static final Logger logger = LoggerFactory.getLogger(MfaService.class);`

Some helpful troubleshooting commands:

* `kubectl exec -it $(kubectl get pod -l app=springboot -o jsonpath="{.items[0].metadata.name}") -- /bin/bash`
* `kubectl logs -l app=springboot --tail=100`

Note to self, you shouldn't instantinate a service in the constructor. That will breaks the autowiring. 

```java
    public MfaService mfaService;

    public MfaController() {
        this.mfaService = new MfaService();
    }
```

Also explictly using `Autowired` is now frowned upon:[^7]

```java
    @Autowired
    public MfaService mfaService;
```

Instead use constructor injection, like so:

```java
    public MfaService mfaService;

    public MfaController(MfaService mfaService) {
        this.mfaService = mfaService;
    }
```

# hcaptcha


# Future Todos

I learned that Java exception are slow and in a web app they should be avoided for performance sensitive code. There's a couple of alternatives for error handling, like returning `nulls`, using `Optional<String>`, error codes, result class. This is something I should explore more and then refactor my code. 

# References
[^1]: *(As of Spring Boot 2.1, we no longer need to load the SpringExtension because it's included as a meta annotation in the Spring Boot test annotations like @DataJpaTest, @WebMvcTest, and @SpringBootTest.)*

[^2]: I don't trust the third-party embedded redis servers. And redis doesn't have a official embedded redis server. 

[^3]: *setting the required attribute to false indicates that the corresponding property is optional for autowiring purposes, and the property will be ignored if it cannot be autowired.* [https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)

[^4]: [https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/mock/mockito/MockBean.html](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/mock/mockito/MockBean.html)

[^5]: The issue stems from multiple env variables colliding, specifically `spring.smokescreen.port=4750`, which causes this error: `Error creating bean with name 'webProxy'... Error creating bean with name 'webClientConfig': ... field 'smokescreenPort': Failed to convert value of type 'java.lang.String' to required type 'int'; For input string: "tcp://10.43.148.106:4750"`. This `@Value("${spring.smokescreen.port}")` tries to load this env variable `REDIS_PORT=tcp://10.43.97.178:6379` instead of from the `appliciation.properties` file. The order of operations states that `5) OS environment variables` comes after `3) properties files`, but for some reason that isn't working. [https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.typesafe-configuration-properties.vs-value-annotation.note](https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.typesafe-configuration-properties.vs-value-annotation.note)

[^6]: *However, when you authenticate with your test credentials, we will not charge your account, update the state of your account, or connect to real phone numbers.* [https://www.twilio.com/docs/iam/test-credentials](https://www.twilio.com/docs/iam/test-credentials)

[^7]: [https://medium.com/@dulanjayasandaruwan1998/spring-doesnt-recommend-autowired-anymore-05fc05309dad](https://medium.com/@dulanjayasandaruwan1998/spring-doesnt-recommend-autowired-anymore-05fc05309dad)
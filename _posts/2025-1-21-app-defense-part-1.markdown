---
layout: post
title: "Application Defense: Part I - Sample App"
date: 2025-1-21
tags: ["app"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement

Build a bunch of application defensive mechanisms. This problem can be further broken down into:

* Step 1: **Build a sample app with Spring Boot**

# Spring Boot Refresher

My sample app will use Spring Boot. It's been a while since I've dove into Spring Boot concepts. I've found that this book by Laurentiu Spilca is an excellent breakdown: [Spring Start Here: Learn what you need and learn it well](https://www.amazon.com/gp/product/B09HJLK2BN/).

# Sample App

Simple Spring Boot app built using Gradle Kotlin DSL. A simple controller and some code that plays around with dependency injection, beans, and annotations. Nothing complex, just need a simple app to get things running. 

* [https://github.com/JacksonKuo/app-springboot](https://github.com/JacksonKuo/app-springboot)

# Book Reading Notes[^1]

A majority of these notes will be pulled straight from the book. My notes are kinda a mess, but just a hogpodge of things that stood out. Haven't read the entire book yet, will continously update as I get further. 

#### Chapter 1 - Intro

* Spring is a ecosystem of frameworks
    * Spring Core
        - Spring context
        - Spring aspect
    * Spring MVC
    * Spring Data Access
    * Spring test module
* Inversion of Control (IoC): Instead of the app controlling the dependencies, it's the dependencies a.k.a Spring framework, controlling the app.
* IoC container = spring context, manages object instances
* AOP = aspect-oriented programming = spring can control instances added to its IoC container, and it can intercept methods that represent behavior of these instances. Spring AOP is often how the framework interacts with your app
* Spring Data Access = module of Spring Core
* Spring Data = independent project in Spring ecosystem
* Spring Boot
    - convention over configuration

#### Chapter 2 - Spring context

* context = application context, place where framework manages all object instances. by default, spring doesn't know of any objects in your app, you need to add them to the context
* build tools: maven, gradle
    - download dependencies
    - run tests
    - syntax
	- security vulns
	- compiling
	- packaging
* object instances = beans
* spring dependency = spring context
* beans
	* @Bean annotation
	* stereotype annotation
	* programatically

> Spring Bean is nothing special, any object in the Spring framework that we initialize through Spring container is called Spring Bean. Any normal Java POJO class can be a Spring Bean if it’s configured to be initialized via container by providing configuration metadata information.[^2]

* `AnnotationConfigApplicationContext`: most used in today's approach

`var context = new AnnotationConfigApplicationContext(ProjectConfig.class)`

* configuration class is a special class that instructs Spring to do certain actions, like create beans

```
@Configuration
public class ProjectConfig

@Bean
Cat cat() {
    return cat
}
```

* define a method that returns the object instance and annotate with @Bean to add to the context
* The method with @Bean, shouldn't use a verb name, like traditionally in Java methods. By convention use a noun to represent the object instance

`@Bean(name = "roxy")`

`@Primary`
* Multiple beans of the same kind, a primary bean is the one that spring will choose if there's multiple options

* stereotype annotations: write less code to add a bean to the context
    * @Component: add the annotation above the class that you need an instance for
    * still have a config class to tell spring where to look for classes annotated with stereotypes
    * can use both @Bean and stereotypes together as well

* @Component: annotates class to add bean
```
@Component
public class Cat
```
* @ComponentScan: tell spring where to look for classes
```
@ComponentScan
public class ProjectConfig
```

* by default, Spring doesn't search for class annotation with stereotype annotations, a.k.a. @Component
* If you're using Spring Boot, it automatically scans for components in the package where the main application class (the one with @SpringBootApplication)
* ProjectConfig is empty, just used to point where @Components are located
```
@Configuration 
@ComponentScan(basePackages = "main")
public class ProjectConfig {
//empty
}
```
* Using @Component, means you don't need write a cat method 
```
    @Bean("BlueCat")
    Cat cat() {
        return new Cat("Orange");
    }
```

* @Bean annotations vs stereotype annotations
* @Bean
    * multiple instances
    * 
        ```
        @Bean
        Cat cat1(){return cat}

        @Bean
        Cat cat2(){return cat}
        ```
    * only used when you can't add the bean otherwise
* stereotype annotations
    * can only add one instance of a class
    * only for classes you own
    * less boilerplate
    * preferred
* @PostConstruct
    * call method after constructor finishes
* registerBean: programmatically depending on specific conditions
    * beanName
    * beanClass
    * supplier
    * customizers

#### Chapter 3 - Wiring Beans

1. @Bean, link beans by calling methods that create them - wiring
2. @Bean, enable Spring to provide us a value using a method parameter - auto-wiring
3. @Component, dependency injection using @Autowired

* implement relationships between beans. we want this so one object can delegate responibility of something to another object
* direct wiring
    * in config class call one method from another

```
    @Bean("BlueCat")
    Cat cat() {
        return new Cat("Orange");
    }

    @Bean("Alice")
    Person person() {
        return new Person("Alice", cat());
    }
```

* the `@Bean cat()` and the `new Person("Alice", cat())`, don't create 2 cats, Spring smart enough to know you're referring to the existing cat bean in the context

* wiring using method parameters
```	
    @Bean("BlueCat")
    Cat cat() {
        return new Cat("Orange");
    }

    @Bean("Alice")
    Person person(Cat cat) {
        return new Person("Alice", cat);
    }
```

* dependency injection. the framework sets the value of an object of an app
* @Autowired, mark an object property where we want Spring to inject a value from the context
    * inject values through field of the class
    * inject values through constructor parameters
    * inject values through setters
* field
* constructors
	* can use final
	* no one can change value after Spring initializes
* setters
    * pretty rare, not really used
* @Qualifier
```
    @Bean("Alice")
    Person person() {
        return new Person("Alice", cat());
    }
```
* if there are multiple cats
* if multiple beans of the same type are available, will choose the bean with the same name.
* only needed when using @Bean, since @Component will only generate
* @Autowired used often with stereotype annotation, a.k.a. @Compoennts

#### Chapter 4 - Using abstractions

* @Override
* Overriding in Java occurs when a subclass implements a method which is already defined in the superclass or Base Class.
* Java annotation

#### Chapter 7 - Understanding Spring Boot and Spring MVC

* 

#### Chapter 11 - Consuming REST endpoints

* OpenFeign
    * spring cloud family
    * only need to write an interface
    * @EnableFeignClients annotation on a configuration class and tell OpenFeign where to search for client contracts
* RestTemplate
    * maintenance mode starting spring 5 and will be deprecated
    * most common, but legacy
    * not easy to call endpoints both asynch and synch
    * HttpHeaders instance, HttpEntity instance, exchange()
    * shouldn't use RestTemplate in new implementations
* WebClient
    * reactive programming
    * RestTemplate documentation gives a recommendation
    * advanced approach
    * strongly coupled to make app reactive
    * nonreactive app
        * a thread executes a business flow
        * request is blocked waiting for I/O operations to finish
        * app creates a new thread for each request and thread executes a step one by one, like a timeline
        * thread is idle while an I/O call blocks... stay and occupy app memory
    * reactive app
        * backlog of tasks and a team of threads solving them
        * all tasks from all requests are on a backlog. any thread can work on tasks from any requests
        * two components: producer and subscriber to implement dependencies
        * a task returns a producer to allow other tasks to subscribe to it
        * a task uses a subscriber to attach to a producer of another task
        * WebClient requires WebFlux
        * Mono, this class defines a producer
* All three use proxy classes, versus what I currently have

#### Chapter 15 - Testing your Spring app

* unit tests and integration tests
    * validation with minimum effort, understand use-case, documentation, early feedback of issues
    * regression testing: constant retesting functionality to validate it still works
    * JUnit in Action [https://www.amazon.com/JUnit-Action-Catalin-Tudose-ebook/dp/B09781KDZG/](https://www.amazon.com/JUnit-Action-Catalin-Tudose-ebook/dp/B09781KDZG/)
    * a test class should focus only on a particular method
    * reason to keep the methods in your application small
    * unit tests: validate a methods logic
    * integration tests: validate method logic and its integration with specific capabilities the framework provides. help when upgrading dependencies
    * unit tests: eliminate all dependencies of the capability they test. covering a isolated piece of logic. valuable bc when one fails, you know exactly where
    * first we write tests for the happy flows, no exceptions or errors
* unit tests:
    * assumptions, call/execution, validations
    * arrange, act, assert
    * given, when, then
* assumptions
    * identify inputs and dependencies
    * dependencies: anything the method uses but doesn't create itself. method parameters, object instances
* mocks via Mockito
* junit 5 jupiter, junit 4 well-establish in legacy projects
    * mock(AccountRepository.class)
    * given(): control mock's behavior
    * @DisplayName annotation: describe test
    * verify(): verify a mock's object's method has been called
    * in many cases, mock() declared inside method
    * @ExtendWith(MockitoExtension.class) 
    * @Mock: injects a mock object in the annotated attribute
    * @InjectMocks: create a object to test, then inject all mocks in it's parameters
    * assertThrows(): check that the method throws an exception
    * Optional.empty()
    * verify(foobar, never())
    * assertEquals()
* integration tests:
    * between two (or more) objects
    * object and framework capability
    * app with persistence layer
    * don't neccessrily have to mock dependencies. can still mock if test doesn't care about that specific service
    * use a in-memory database such as H2, use real db can cause latencies
    * in a spring app, generally use integration test sto verify app's behavior correctly interacts with the capabilities Spring provides
    * spring integration test: enables spring to create beans and configure the context
    * @MockBean (springboot annotation, not spring)
    * can you @Autowired to inject the object
    * if we upgrade spring version and DI no longer worked, test would fail
    * use unit test to validate components and spring integration test to validate integration scenarios. don't use spring integration tests to validate component's behavior. waste of time and resources

# References

[^1]: [Spring Start Here: Learn what you need and learn it well](https://www.amazon.com/gp/product/B09HJLK2BN/)

[^2]: [https://www.digitalocean.com/community/tutorials/spring-ioc-bean-example-tutorial](https://www.digitalocean.com/community/tutorials/spring-ioc-bean-example-tutorial)
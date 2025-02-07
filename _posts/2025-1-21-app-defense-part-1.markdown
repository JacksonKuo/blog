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

* `AnnotationConfigApplicationContext`

```
@Configuration
public class ProjectConfig
```

`var context = new AnnotationConfigApplicationContext(ProjectConfig.class)`

* configuration class is a special class that instructs Spring to do certain actions
* define a method that returns the object instance and annotate with @Bean to add to the context
* The method with @Bean, shouldn't use a verb name, like traditionally in Java methods. By convention use a noun to represent the object instance

`@Primary`
* Multiple beans of the same kind, a primary bean is the one that spring will choose if there's multiple options
* stereotype annotations
* annotation class to add bean
```
@Component
public class Cat
```
* tell spring where to look for classes
```
@ComponentScan
public class ProjectConfig
```

* by default, Spring doesn't search for class annotation with stereotype annotations, a.k.a. @Component
* If you're using Spring Boot, it automatically scans for components in the package where the main application class (the one with @SpringBootApplication)
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
    * only used when you can't add the bean otherwise
* stereotype annotations
    * can only add one instance of a class
    * only for classes you own
    * less boilerplate
    * preferred
* @PostConstruct
    * call method after constructor finishes
* programmatic approach
* registerBean
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

# References

[^1]: [Spring Start Here: Learn what you need and learn it well](https://www.amazon.com/gp/product/B09HJLK2BN/)

[^2]: [https://www.digitalocean.com/community/tutorials/spring-ioc-bean-example-tutorial](https://www.digitalocean.com/community/tutorials/spring-ioc-bean-example-tutorial)
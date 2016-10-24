---
id: ghost-8
title: Tidy Spring - Starting a Project
author: Grzegorz Ziemonski
summary: Spring has become extremely popular among Java developers. After all, it's a great project with tons of useful features. I decided to share with you a list of practices I use to work effectively on Spring applications, avoid unnecessary framework coupling and achieve a tidy application architecture. The whole set of practices was too long for one article so I divided it into smaller parts. In this one, I will focus on things related to starting a new Spring application.
date: 2016-04-21
tags:
    - java
    - spring
---
Spring has become extremely popular among Java developers. After all, it's a great project with tons of useful features. I decided to share with you a list of practices I use to work effectively on Spring applications, avoid unnecessary framework coupling and achieve a tidy application architecture. The whole set of practices was too long for one article so I divided it into smaller parts. In this one, I will focus on things related to starting a new Spring application.

# Start from the business
If you're just starting the project, I recommend starting from writing some business features. You don't need to configure anything related to Spring right now. Just write some classes and abstract everything you need Spring for. Once you have enough business code to deliver something, you get to decide. If you still feel like Spring fits your project, it's the right time to add it.

## But I want to see the app working!
Many people want to start with some basic Spring configuration to be sure that they're building a working product. I think we should avoid this way of thinking. The fact that you're writing the right thing, in the right way should be expressed as a set of acceptance and unit tests. Good tests and green IDE bars mean it works.

This approach to starting a project has a huge benefit. Using it implies that your code is framework independent, which, at least in my opinion, is a good thing. And you remain free to change your mind about the framework until you actually have to deliver.

If you're still not convinced, I recommend reading one of my previous posts: [Framework Coupling](http://tidyjava.com/framework-coupling/). From now on, I will assume that you made a deliberate decision to use Spring in your project.

# Architecture overview
Even though, I don't believe in architecture diagrams, you should have a clue what are the major parts of the application. For this purpose, I created a very simple diagram, how the high-level architecture should look like for a typical Spring web application with persistence:
![](/img/arch.png)
The green parts are the ones that we can use Spring for. In these components we can use Spring mechanisms directly. "Web" groups components related to our web interface (MVC/REST). "Main" is responsible for context configuration, creating business beans and injecting their dependencies. "DB" is a persistence component.

The red part represents components with business features. Business components should not be dependent on Spring or any Spring-related (green) components. I recommend putting them in separate Maven/Gradle modules with no "green" dependencies.

# Spring Boot
If there's no good reason against it, I would use [Spring Boot](http://projects.spring.io/spring-boot/) for setting up Spring in my projects. It's very easy to configure, all you have to do is add the dependency and a few annotations. It has fantastic IDE integration - you can run your app via main method, use live reload to avoid restarting the whole application for a small class change and more. Also, it has some fancy features that we can't have in custom setups e.g. if we use it's parent pom, we get preconfigured dependency versions, which guards us against incompatibilities.

# Java over XML
I see several benefits of using XML configuration - we avoid annotation hell, Spring configuration is (somewhat) centralized and there's no need to recompile anything when configuration changes. Unfortunately, XML looks extremaly human-unfriendly (that's subjective, but I haven't found a person who likes it). Long XML files are a PITA. Tons of short XML files can be an even bigger one.

Java, on the other hand, is easy to read - as Java developers, we're used to it. Annotations in Spring-related classes like controllers are very handy. With no work, we can reduce the number of beans in our context, which makes it easier to manage (more on that in a future post of this series). Also, there's a general shift towards Java configuration in Spring community, I don't see a point going against it.

# Config Classes
Once we decide for the Java config, we will have to put our Spring configuration into classes. The last thing we want to do is put everything in a single class. For Spring configuration, we apply the same rules that we use for other classes. Things that are related, aka "change for the same reason", should go into a single class. Things that are totally unrelated e.g. controller configuration and database configuration, surely have to go into separate classes. If you decide to go with Spring Boot, then the class containing `main`, should not contain any other configuration (apart from annotations above it, of course).

# (No) Packaging
I don't really like starting with a predefined package structure. Some sources recommend creating some standard packages for every application e.g. "domain", "controllers" or "services". It's likely that packages like these will emerge from the class structure in the future, but I believe packaging should be reactive. Write some classes, then segregate them. Don't lock yourself to any structure at start, just because it has worked for someone else before.

# Conclusion
That's it for part one. We have seen that a typical Spring application should have a few Spring-related components and the business part, which doesn't even know Spring exists in the project. For configuring Spring in a new project, Spring Boot with Java Configuration is the most effective way to go. Each kind of Spring configuration goes into a separate class - good programming rules still apply! Last, but not least, we should not start with predefined packages. Package structure should emerge from our classes further into implementation.

### Other parts of the series:
[Tidy Spring - Configuration Properties](http://tidyjava.com/tidy-spring-configuration-properties/)  
[Tidy Spring - Services](http://tidyjava.com/tidy-spring-services/)
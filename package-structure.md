---
id: ghost-18
title: Package Structure
author: Grzegorz Ziemonski
summary: Packaging is one of the underrated features of Java. We use common sense to put things here and there, after some time everybody knows where certain things go and nobody cares any more. Well, as long as it doesn't slow you down, it doesn't shock newcomers and everybody on the team is fine with it, you should be just fine. Nothing wrong will happen. In the end, IDEs are so smart that it only takes a few clicks to reorganize the whole project. But we can do better! We can really benefit from a good package structure.
date: 2016-06-30
tags:
    - java
---
Packaging is one of the underrated features of Java. We use common sense to put things here and there, after some time everybody knows where certain things go and nobody cares any more. Well, as long as it doesn't slow you down, it doesn't shock newcomers and everybody on the team is fine with it, you should be just fine. Nothing wrong will happen. In the end, IDEs are so smart that it only takes a few clicks to reorganize the whole project. But we can do better! We can really benefit from a good package structure.

### Naming Consistency
You don't have to start with a reversed domain. You should, but you don't have to. You can use camelCase, you can even use underscores and numbers! Only one thing is important: **CONSISTENCY**. Decide on a certain convention for the project and stick to it. Everybody\_hatesInconsistencies\_or\_atleastIdo.

### Not About Object Types
Most package structures are organized around object types e.g. controllers, services, exceptions, enums, wtfs. This, of course, gives use some organization, but let's face the truth: I don't need my `SuperUsefulException` to be in `exception` package to know that it's an exception and hiding my `SuperUsefulException` between 20 others doesn't look too organized.

### Expressing the Architecture
The package structure should be organized around the architecture. If I have a "Usefulness" component/microservice, then I'll start my package tree with `usefulness` (of course, if I want my reversed domain in there, I put it before). If I claim this component has a three layer architecture with resources layer, business layer and database access layer, I better have `resources`, `business` and `database` packages just after the component name. This is the first step to ensure that the architecture is in the code, not just on the diagram made by some architect who stopped coding 2000 years ago, because the management wanted him to.

In practice things ain't always that easy, but you get the idea. Let's look at the diagram I stole from [Testing Strategies in Microservices](http://martinfowler.com/articles/microservice-testing/):
![](/img/stolen.png)
What packages would this application have? Well, at least `resources`, `services`, `domain` and `database` (I like this name for keeping all db related stuff). Then, depending on where I prefer to keep my interfaces, I might also have `repositories` and `gateways` packages. In case of implementing the gateways, I'd probably go for a separate package for each of external services. These external service models, configurations and stuff probably won't fit in any of the existing packages.

### Use Cases in the Deep
Deeper down in the package tree, I might want some more organization i.e. not all certain layer related classes should go into one package, at least for a reasonably big application. I'm still not going for type-based packaging. I'd rather prefer packaging around use cases or groups of use cases. As I said before, I don't need an `exception` package to know that a class is an exception, but having `SomeUsefulException` in the same package as `SomeUsefulService` can tell me that these two classes are related without reading the code. By seeing `FullOfGoodStuff` interface next to that service, I can conclude that the service depends on it. You see, we want cohesion in there, we want the packages to be **SID**.

### Single Responsibility Principle
Classes that are used together to achieve the same goal e.g. a service, it's request and response, and exceptions it throws, should be kept in the same package. In general, things that change for the same reasons should be kept close to each other. This helps us reason about the application, without diving deep into implementations of certain classes. Of course, the object-type based packaging is the exact opposite of this.

### Interface Segregation Principle
That's not always possible, but only certain classes, containing package's "public API", should be declared public. Others, if possible, should be kept package local. This way, noone can depend on classes we don't want them too. Why would someone throw `SomeUsefulException` from a class that isn't related to "something useful"? We will get more possibilities of restricting access to classes in Java 9 thanks to Project Jigsaw and I really look forward to using it.

### Dependency Inversion Principle
We say that package `A` depends on package `B` if any class in package `A` imports any class from package `B`. We want this dependencies to point in the right direction e.g. database package being dependent on `services` or `repositories` package, not the other way around. For this rule to be correctly applied to packages, all classes in the package have to follow DIP themselves. By [analyzing dependencies](https://www.jetbrains.com/help/idea/2016.1/analyzing-dependencies.html) of packages, we can look for potential DIP violations in the project.

### Wrapping Up
Bad packaging ain't deadly, it won't kill your project, but doing it well can help you reason about the application. Be consistent in naming packages or people will be angry. Express your architecture and use cases using packages. Classes that change for the same reasons should end up in the same package. Classes that don't expose a public API towards other modules/packages should not be public, don't let people depend on them. Guard [dependency inversion](http://tidyjava.com/dependency-inversion-in-java/) on package level to make sure that all classes follow DIP too.
---
id: ghost-12
title: Tidy Spring - Services
author: Grzegorz Ziemonski
summary: Spring has become extremely popular among Java developers. After all, it's a great project with tons of useful features. I decided to share with you a list of practices I use to work effectively on Spring applications, avoid unnecessary framework coupling and achieve a tidy application architecture. The whole set of practices was too long for one article so I divided it into smaller parts. In this one, I will focus on **services**.
date: 2016-05-12
tags:
    - java
    - spring
---
Spring has become extremely popular among Java developers. After all, it's a great project with tons of useful features. I decided to share with you a list of practices I use to work effectively on Spring applications, avoid unnecessary framework coupling and achieve a tidy application architecture. The whole set of practices was too long for one article so I divided it into smaller parts. In this one, I will focus on **services**.

### What is NOT a (good) service?
If you've read my other texts, you might have noticed that I'm mostly targetting bad practices present in many applications. This one is no exception. It's not uncommon for a Spring application to contain a class like this:
<script src="https://gist.github.com/tidyjava/023142e38915ccfa7dd08c32fc85c825.js"></script>

Then, every other method related to MyObject, we can think of, goes into this service. And, of course, every business object type gets it's own service like this. Once the application grows beyond basic CRUD, people try to make use of their "nice" services together and end up mixing them into spaghetti:

![Spaghetti](/img/spaghetti.png)

I could write a lot about why it's a bad idea, but I'll leave that to you and focus on the correct approach instead.

### What is a (good) service?
A service is a class representing a single application use case or a part of it. Therefore, we say that it contains application specific business rules (think Interactors in [The Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)). For the most part, it creates and orchestartes other objects to fulfill use case's requirements.  Service classes belong to the ["business part"](http://tidyjava.com/tidy-spring-starting-a-project/) of the application. This means they are free of Spring dependencies (but still are a very important part of Spring applications).

![Service](/img/service.png)

### How to name a service?
By the name of the use case, of course! If you're implementing the process of placing an order for an online shop, then name it `PlaceOrder` or `OrderPlacement`. Anything that describes the process behind the use case, and only it!

There's a great gist about naming service objects in Ruby, that applies to Spring services as well, [here](https://gist.github.com/blaix/5764401) (read it, seriously!).

Also, there are certain words we want to avoid like "Service" or "Manager". These words add no value and work like a magnet. Imagine what could happen if we named the order placement service an `OrderService`. What do you think, where will cancelling an order go? Where will seeing order details go? All `OrderService` or `OrderManager` says about the class is that it contains *something* related to orders. A name has to be descriptive and precise.

### Input and Output
Since services are near the boundary of the "business part", they should use simple data structures as their inputs and outputs. A map of Strings, a class with public fields or private fields with accessors might work. The reason for this is that we want to have all business rules closed inside the business component. Therefore, domain objects should not leak outside. It is service's responsibility to convert the data structure to domain objects and backwards (of course, it might use a helper class for this if needed). This idea has it's roots in [Hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture) and [The Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html).

### Stateful or stateless?
It depends. In most cases, a stateless singleton bean is enough. If it isn't, there's nothing wrong with making it stateful. I've seen a few applications organized around only stateless services/beans and, in some cases, it works really badly. If the obvious implementation of the service is stateless, so be it. But once it begs for some state e.g. many arguments are passed down through service's methods, you should make it stateful. Btw. This applies to all beans, not just services.

### How to create a service instance?
Since I'm assuming that business classes have no knowledge of the framework, then obviously I can't use any Spring annotation like `@Service` or `@Component`. But there are a few other options:
 
* `new` in the controller - might seem like some nasty coupling, but could be enough in some cases e.g. a simple service with no dependencies
* `@Bean` inside a `@Configuration` class in the main component - for stateless services
* Factory class annotated with `@Component`, used inside the controller - for stateful services

When creating a service, we inject dependencies using a constructor or a bunch of setter methods. Apart from what many people say, I think there isn't much advantage of using constructor injection instead of setters, especially if you have a lot of dependencies. On the other hand, a lot of dependencies might indicate a problem with the design.

### Example
Let's have a more detailed look at the order placement example. We'll assume that processing a new order consists of a few steps:

1. Saving the order.
2. Creating an invoice, associated with the order.
3. Sending the invoice to the customer.
4. Sending a parcel to the customer.

Of course, our service does not have to implement all of these things by itself. Instead, it will orchestrate it's collaborators:

![Orchestration](/img/orchestration.png)

This diagram might (make it) seem complicated, but the code could be very simple:

<script src="https://gist.github.com/tidyjava/54b17c9bab6569170e89e217db690d61.js"></script>

The service seems fine with being stateless, so we can create it as a singleton bean in a `@Configuration` class:

<script src="https://gist.github.com/tidyjava/0f2ec972186f1d54a1d2b8a2704d08ed.js"></script>

As the last step, we autowire the bean in our controller:

<script src="https://gist.github.com/tidyjava/72c1e3d451829d81eb3062a6501c2dd7.js"></script>

Please note that the controller *can*, but *doesn't have to* have the same input as the service.

### Partitioning
We can imagine that there's much more to the process of placing an order than just these few steps. And, of course, each of these might be much more complicated e.g. it might require more operations, logging or updating business metrics. In such case, it would be way too much to implement in a single service (or not, you've read [the gist](https://gist.github.com/blaix/5764401) I linked before?). We could fix that by splitting the process into smaller services, that will become direct collaborators of the original service:

![Orchestration 2](/img/orchestration-1.png)

An obvious benefit of (and another reason to perform) such partitioning is that the smaller services are likely to be reusable.

### Conclusion
Services are about use cases. They orchestrate other objects/services in order to fulfill requirements behind a use case. A good name for a service is the name of the process behind the (part of) use case it represents. A service consumes and produces simple data structures, thus protecting the business logic from leaking outside the boundaries. It can be either stateful or stateless, which will impact the way we create it. Once we find out that our service is too big or we want to reuse a part of it, we can split the service into smaller ones.

##### Other parts of the series:
[Tidy Spring - Starting a Project](http://tidyjava.com/tidy-spring-starting-a-project/)  
[Tidy Spring - Configuration Properties](http://tidyjava.com/tidy-spring-configuration-properties/)
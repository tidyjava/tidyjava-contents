---
title: Overengineering
author: Grzegorz Ziemonski
summary: Reading a lot of stuff online and talking to people, I feel like there's a notion that the `interface` keyword is a sign of overengineering, legacy code and the old, complicated way of writing code in general. Microservices however... a completely different story! So small, simple, elegant... and powerful! Look at this scaling! Look at this fault tolerance! Look at this, look at that!
date: 2016-11-01
tags:
    - java
---
Reading a lot of stuff online and talking to people, I feel like there's a notion that the `interface` keyword is a sign
of overengineering, legacy code and the old, complicated way of writing code in general. Microservices however... a
completely different story! So small, simple, elegant... and powerful! Look at this scaling! Look at this fault tolerance!
Look at this, look at that!

I'd like to propose a thinking exercise. Imagine that you're working on a fairly sized enterprise project. Nothing
special, yet another e-commerce, ERP or whatever. The load is perfectly sustainable for a single instance, no need for
massive scaling etc. You noticed that the part of the project you've been working on has grown significantly and after
a quick analysis, you discovered 2 distinct responsibilities to be separated.

![Overengineering - Before](/img/overengineering-before.png)

Let's assume that `responsibilityA` depends on `responsibilityB`, both contain significant amounts of code and,
from now on, two different teams should be able to work on them independently.

# Overengineering
We'll start by overengineering the integration. The two teams meet together and work out a contract, which is represented
by a regular Java `interface`, documented with JavaDoc. All the messages sent via the interface are JDK types or simple
data structures (think: POJO with no methods besides accessors).

![Overengineering - After](/img/overengineering-after.png)

The team responsible for A stuff is only allowed to use classes from `responsibilityB.api` package. We could force that
by e.g. moving the package into a separate Maven module, but it's out of this article's scope.

# Microservices
Now we'll change this medieval, overengineered bullshit into simple, scalable microservices. Again, the two teams meet
and agree on a contract. This time, they'll use REST and JSON, documented by Swagger and protected by an [integration
contract test](http://martinfowler.com/bliki/IntegrationContractTest.html).

![Overengineering - Microservices 3](/img/overengineering-microservices3.png)

Please note that necessary configurations like web client, service registry, circuit breakers etc. were left out.
And we didn't even touch on infrastracture and all configurations required for separate deployments.

# Thinking time!
I don't want to lay out a conslusion for you. Instead, I'll ask you a few questions:

* Which solution is more complex at this point?
* Which solution would be more complex with full-blown implementation (i.e. after splitting the first one into Maven
modules or after adding all necessary stuff to deploy 2 separate microservices)?
* What are pros/cons of each solution in general?
* Which of the pros/cons apply to your projects?
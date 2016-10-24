---
id: ghost-11
title: Domain Object Persistence
author: Grzegorz Ziemonski
summary: Persistence is a part of almost every Java enterprise application. Unfortunately, persisting domain objects is a non-trivial task. On the contrary, it generates a lot of problems, especially when we're [using a relational database](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) [^1]. Let's see different approaches to the problem.
date: 2016-05-05
tags:
    - java
    - architecture
---
Persistence is a part of almost every Java enterprise application. Unfortunately, persisting domain objects is a non-trivial task. On the contrary, it generates a lot of problems, especially when we're [using a relational database](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) [^1]. Let's see different approaches to the problem.

### All-in-one model
The simplest approach, present in most applications, is to use the same model for persistence and business purposes (often in other layers too). From architectural perspective, we get something like this:

![](/img/all_in_one_dependencies.png)

If we consider the fact that these entity classes most likely contain some annotations regarding the database (like JPA), then we make our whole application dependent on database details. Once we update anything in the database schema, we have to update our business classes. Problems related to ORM mechanisms are spread throughout the application e.g. we might get a lazy-init exception in our web controller. What's worse (from domain perspective), developers often add validation annotations to their entities to ensure that data written to the database is correct. At this point, people are extremaly reluctant to add business functionalities to these classes, because they already contain a lot of code related to the database. Let's add a bunch of getters and setters (some mapping tools require that) and we get a recipe for database centric architecture and [anemic domain model](http://www.martinfowler.com/bliki/AnemicDomainModel.html). No good, no good sir!

![](/img/database_centric.png)

Don't get me wrong. I'm not saying that all those bad things are always the case. I just think it's very hard to keep the model fit and avoid other problems with this solution, especially in big projects. And **once these problems start to appear, this approach is no longer the simplest**!

<script src="https://gist.github.com/tidyjava/255389a70dccdc4ca6c0e8a654c4df40.js"></script>

Such class used all around the codebase is definitely a code smell.

### Separate models
The next simplest solution, when we take into consideration all the problems mentioned above, is to separate models for the database stuff and the domain. The dependencies should look like this:

![](/img/separate_model_dependencies.png)

Now, the database layer is responsible for mapping database entities to domain objects and the other way around. The domain part doesn't know anything about the database stuff, which is definately a good thing. We're also free from ORM related problems, like lazy-init exceptions, because we map all important information before it gets out of the database component.

In the code, our models might look like these:

<script src="https://gist.github.com/tidyjava/8b247e3c1a273559085a60dbefa6fbe4.js"></script>

The obvious downside of this approach is that we have to add a lot of boilerplate mapping code like this:

<script src="https://gist.github.com/tidyjava/59bfc8c67e71bdf6a901852b98792397.js"></script>

If we have a lot of classes with a lot of state, this can be painful (or rather boring, and I strongly discourage using any magic automappers for this process). On the other hand, complex mappings like enums to strategies or collections are pretty straightforward in this solution. Also, if we use a builder instead of setters, our encapsulation improves a lot. Nobody can change our object's state in any uncontrolled manner.

### Model inheritance
We might decide that these extra mapping classes and separate models are making our application too complex. In such case, we can use inheritance to achieve proper domain-database separation. Look at the diagram:

![](/img/inheritance_dependencies.png)

Now, we use the database layer to provide all required state for our domain object. Since the domain part doesn't have state[^2] any more, there's nothing to map!

Look at the code, enjoy the beauty of pure, abstract business class and a poor database servant:

<script src="https://gist.github.com/tidyjava/23d6b6ac4f34056cc78cf13c5f8742bd.js"></script>

It looked very strange to me, when I first saw it. But it works and, in fact, works very well. We have our separation and we don't have any boilerplate code. From the encapsulation point of view, we don't have to expose anything as public until we really need to. Perfect solution?

Not really. Now we're exposed to ORM-related problems again, because our domain objects are JPA entities under the hood. Also, it can be pretty tricky to effectively use mappings like collections or enum-to-strategy in this setup. It's not impossible, but definitely harder, which may make our colleague developers (or ourselves) reluctant to deal with the problem.

### Conclusion
It's all about discipline and complexity. In general, we should strive to keep our domain model as database independent as possible. Separate domain and database models generate least problems and make mapping pretty straightforward, but require a lot of boilerplate code. Inheritance, on the other hand, seems convenient to use, but can cause same problems as all-in-one model if used incorrectly. In the end, none of these solutions forces clean separation, they just *enable* it.

### Extra reading
When doing research for this post, I ran across many interesting sites, you might want to read. Here's some of them:

* [Active Record vs Objects](https://sites.google.com/site/unclebobconsultingllc/active-record-vs-objects)  
* [ORM is an Offensive Anti-Pattern](http://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html)  
* [15 Tips on JPA Rich Domain Modelling](http://www.adam-bien.com/roller/abien/entry/10_tips_on_jpa_rich)  
* [How should enties be created](https://groups.google.com/forum/#!profile/clean-code-discussion/APn2wQeQiZYwEcfL-TGnPk_h5z7cslG6l1dHPmS2_B_8ciN3dggJHPN5eR4q4UwDbfn01Aw4Rc8M/clean-code-discussion/UzxMB39w15M/PqUNA6Wvf9UJ)  
* [Clean Architecture - Entities - People are Confused?](https://groups.google.com/forum/#!searchin/clean-code-discussion/od$3A$20unclebob/clean-code-discussion/mvP_NR2MUPc/wrkHHxzkDQAJ)  
* [Anemic Domain Model](https://groups.google.com/forum/#!searchin/clean-code-discussion/od$3A$20unclebob/clean-code-discussion/FlZq3EWiFNU/1_R5tmgFn-IJ)  
* [Clean Architecture: too many layers of indirection?](https://groups.google.com/forum/#!topic/clean-code-discussion/GKJhVkZeTTY)  
* [Basic Java Persistence API Best Practices](http://www.oracle.com/technetwork/articles/marx-jpa-087268.html)

Also, in many places I have found references to DDD and PoEAA books, which I have to read myself :)

[^1]: First time you read that wiki page, you might get a feeling that it's impossible for OO and RDBs to work together :D
[^2]: Technically, the object can have concrete state and we can add JPA annotations to getters/setters in the DB derivative, but many sources suggest that annotating fields is a better approach to using JPA in general.
---
id: ghost-14
title: Spring Naming Sucks
author: Grzegorz Ziemonski
summary: Recently, I've been working on a new project using Spring. It's been a great chance to try out new things, gather all the best practices and see what comes out of it. One of the things that struck me was how confusing Spring naming is when trying to go the *right way*.
date: 2016-05-26
tags:
    - java
    - spring
---
Recently, I've been working on a new project using Spring. It's been a great chance to try out new things, gather all the best practices and see what comes out of it. One of the things that struck me was how confusing Spring naming is when trying to go the *right way*.

Let's say I want a class responsible for creating instances of my business classes. Something like:

    public class A {
        public MyBusinessService myBusinessService() {
            return new MyBusinessService();
        }
    }

What would be an appropriate name instead of A? I'd say something like `MyBusinessFactory`. Let's add some spring annotations:

    @Configuration
    public class A {
        @Bean
        public MyBusinessService myBusinessService() {
            return new MyBusinessService();
        }
    }

And it seems more like `MyBusinessConfiguration`. What exactly am I *configuring* right there? Well, you might say it's a context configuration. Then why isn't that named `@ContextConfiguration`? Ah, this one is reserved for integration tests! Actually, it's possible to use `@Bean` in classes with other annotations like `@Component`. But it doesn't help either. That single class is surely not a whole `MyBusinessComponent`!

Btw. What's a `@Component`? Spring annotation for declaring a bean. What's a *component*? Some part of software, in most cases bigger than a single class. Where the heck did they get that name from? Let's check the docs.

> Spring provides further stereotype annotations: @Component, @Service, and @Controller. @Component is a generic stereotype for any Spring-managed component. @Repository, @Service, and @Controller are specializations of @Component for more specific use cases, for example, in the persistence, service, and presentation layers, respectively. Therefore, you can annotate your component classes with @Component, but by annotating them with @Repository, @Service, or @Controller instead, your classes are more properly suited for processing by tools or associating with aspects. For example, these stereotype annotations make ideal targets for pointcuts. It is also possible that @Repository, @Service, and @Controller may carry additional semantics in future releases of the Spring Framework. Thus, if you are choosing between using @Component or @Service for your service layer, @Service is clearly the better choice. Similarly, as stated above, @Repository is already supported as a marker for automatic exception translation in your persistence layer.

Wooow. Seems like every bean is a "Spring-managed component". We just have to live with this notion. Let's keep going.

`@Repository` and the whole set of `*Repository` classes are a "lovely" thing. Repository pattern lets us access our domain objects using a collection-like interface. As far as our domain classes are the ones being persisted, the whole thing looks good. I have a `Person` domain object and a `PersonRepository` with the Spring magic. Things get funny when I decide to have different classes for my domain and persistence. Let's say that I have a `Person` class and it's db counterpart `DbPerson`. Now, in the business layer I am accessing `Person` objects via `PersonRepository` which is implemented using `DbPerson` and... `DbPersonRepository`! The first one is an interface on my business side and the latter is the Spring magic. Repositoryception! My first solution was to shift away from Spring-like naming and make a `DbPersonRepository` that `implements PersonRepository` and uses `DbPersonDao` (here's the Spring magic). That didn't work out well, because my colleagues came along asking what's the difference between a Repository and a Dao in our project and why it's the Dao that extends the Spring `Repository` interface. Grrr.

Last thing I want to hit in this post is `@ConfigurationProperties`. This one seems like a pretty good name. It's related to *configuration* (the intuitive one) and it's related to properties. But.. how am I supposed to name classes using this annotation?! `SomethingConfiguration` suggests that's a `@Configuration` class. `SomethingProperties` might be confusing when used on the business side. `SomethingConfigurationProperties` is a monster! Another thing is, why do I have to add stereotype annotations to these classes? Is there anyone who have ever used `@ConfigurationProperties` without a stereotype annotation?

## @Component
## @ConfigurationProperties
## @WhySpringWhy

---
To not leave you without any solution, I see two ways to go:

* use Spring-like naming everywhere - at least people familiar with Spring will understand it
* use common-sense naming and accept the fact that your `*Factory` classes are annotated with `@Configuration` and `*Configuration` classes aren't etc.

I don't know why, but the latter seems better to me.
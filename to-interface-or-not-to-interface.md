---
id: ghost-17
title: To Interface or Not To Interface
author: Grzegorz Ziemonski
summary: How do you decide if a class should be hidden behind an interface or not? What has to happen to make you think *"Oh, I should create an interface here"*? Quick googling and we land on a [StackOverflow page](http://stackoverflow.com/questions/1686174/when-should-one-use-interfaces) that gives us a handful of hints.
date: 2016-06-23
tags:
    - java
---
How do you decide if a class should be hidden behind an interface or not? What has to happen to make you think *"Oh, I should create an interface here"*? Quick googling and we land on a [StackOverflow page](http://stackoverflow.com/questions/1686174/when-should-one-use-interfaces) that gives us a handful of hints.

First thing there, which I hear, read or see most often is that an interface can have multiple implementations with different behaviours and stuff. I've even read a few times that you *shouldn't* use an interface if there aren't multiple possible implementations. Nonsense! Firstly, interfaces (abstractions in general) give us the power of [Dependency Inversion](http://tidyjava.com/dependency-inversion-in-java/). That's probably even more important than polymorphism. It's obvious that many of such "inverting" interfaces will have only one implementation and that's not a problem! Secondly, there are cases when the possibility of multiple implementations lead to an unnecessary interface. Look and suffer:

    public interface Mapper<S,T> {
        T map(S source);
    }

You have no idea how annoying such interface is in a huge project. Clearly, the number of possible implementations is not the key to the decision.

Next up, we have the ability to mock interfaces to facilitate unit testing. Well, mocking tools have become so powerful that it doesn't matter if it's an interface, abstract class or a static method any more. Not saying that we should use these capabilities, but the argument is, again, not enough.

Last example on that SO page mentions interchangeability (I love this word) of implementations e.g. different data sources or some kind of a strategy. That's a nice thing to have, but not a case every time e.g. when we intentionally have only one implementation.

I also mentioned dependency inversion as a reason to create an interface. That's a bit connected to the point with decoupling and mocks on the SO page, but mocking isn't the only reason to decouple. Dependency inversion is a pretty good motivation to use an interface, but of course, not every dependency has to be inverted and not every interface is meant to do that (even though it does).

Then, what's the key? How to wrap it all together? I propose a rule like this:
==Use an interface when you don't want to care about it's implementation(s).==

I don't want to care, which implementation is used if there's a lot of them and any of these is fine. I don't want to care about the actual class, when I want to mock something (which I want around architectural boundaries btw.). I don't want to explicitly handle every single implementation of a strategy. I surely don't want to care about low level details of an external service connection, database communication or whatever else unimportant.

One more thing that annoyed me recently. If I have a `SomethingRepository` interface, it means I don't care how it's implemented. It doesn't mean I'm using Spring Data or anything similar. It means **I DON'T CARE**.
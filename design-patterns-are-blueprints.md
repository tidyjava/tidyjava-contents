---
id: ghost-13
title: Design Patterns Are Blueprints
author: Grzegorz Ziemonski
summary: Recently I was checking DZone for some interesting programming articles. One of the front page articles stated: [Design Patterns Are Not Blueprints](https://dzone.com/articles/design-patterns-are-not-blueprints). If you haven't read it already, do it now...
date: 2016-05-19
tags:
    - design-patterns
---
Recently I was checking DZone for some interesting programming articles. One of the front page articles stated: [Design Patterns Are Not Blueprints](https://dzone.com/articles/design-patterns-are-not-blueprints). If you haven't read it already, do it now.

So, of course, I got interested and read the text. It fell under the category of I wrote some nice code and I'm proud of it. But I would also argue that it falls under attacking well known, battle tested programming practices (one of many *TDD is bad*, *Design Patterns are bad*, *OO is bad*, etc.).

Immediately I sense some objections. The author didn't outright say "Design Patterns are bad!"; he just mentioned that you don't have to apply them left and right. Attacking articles tend to use words such as *bad*, *sucks*, or *anti-pattern*!

That's all well and good, but it brings me to the title of this article: "Design Patterns Are Blueprints," which the author of the opposing article contested. A pattern is a well-tested, working way of solving a design problem. It should only be used in that kind of a problem—not everywhere possible. Also, to say that anything related to the pattern is its implementation is as silly as doing anything else without thinking.

In his case, it would be stupid to define a `WiringStrategy` interface with a `wire()` method because we would have to add a lot of classes for nothing. The top-level logic would be cluttered by a list of `WiringStrategy` instances and overly generic algorithms.  We would misuse a pattern to get some artificial advantage over the original solution.

Why wouldn't the pattern help? Because the author's case wasn't one we need a strategy for; he didn't need algorithm interchangeability or algorithm-caller separation. We'd be messing with a list of strategies instead of some simple code. (Worst case, he could have really messed himself up with softcoding and externalized the list of strategies to a configuration file. Good for him that he didn't do that!) So you should use the pattern only if it fits your case, not in every similar case.

When do we give up on using a pattern? When someone looking at and maintaining the code would see simple method calls with obvious purposes and understand the priority between them without any patterns. Shifting to separate classes and some fixed structure in the name of flexibility would cause the poor maintainer to go multiple places to understand what the code is doing when it just doesn't seem worth it.

That article may seem like a separated example, but I've personally witnessed a lot of cases that are this bad. Sometimes it's the person who is convinced that design patterns are not blueprints, but misusing a term and propagating misunderstandings on a popular site for programmers is somehow OK. Design patterns have their place and should not be overused, but we should not be redefining well-known and settled terms just because we feel like it. Other times, it's the person who feels the need to object some well-known truth and prove that he's smarter; but I don't think it was the case this time.

At the end of the day, all of our design patterns and best practices are designed to train our minds so that we write good code. They are not supposed to be a substitute for thought, but we should not redefine them unless we have a very strong reason to do so. Also, the best users of design patterns often label their classes `AbcFactory`  or  `XyzStrategy`  or  `AbstractProxyVisitorImpl` because they want to express intentional use of a design pattern (even if it's clearly visible).

Like the wise programmer says: *Not only simplest thing that works. Simplest good design that works!*

PS. This text is not meant to attack anyone personally. I just don't like when people are writing things based on false premises.
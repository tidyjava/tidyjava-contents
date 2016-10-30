---
id: ghost-35
title: Reusable Components in Java
author: Grzegorz Ziemonski
summary: I can't name exact sources, but I feel like ever since I started learning how to program, I was reading tales and legends about "reusable" components, modules and so on. It was not until recently, that something clicked in my head and I found this concept pretty basic. In this text, I'll try to shed some light on creating components and making them reusable. Obviously, it won't be any close to complete, rather an inspiration to explore further.
date: 2016-10-29
tags:
    - java
    - components
    - blogging-platform
---
I can't name exact sources, but I feel like ever since I started learning how to program, I was reading tales and legends about "reusable" components, modules and so on. It was not until recently, that something clicked in my head and I found this concept pretty basic. In this text, I'll try to shed some light on creating components and making them reusable. Obviously, it won't be any close to complete, rather an inspiration to explore further.

### What is a component?
You can find a lot of definitions and approaches to defining a *component*. For the sake of this article, I'll define it like this:

> *Component* is a class or bunch of classes with cohesive responsibilities that form a larger whole.

Now, for some of you this will be valid, some may say that I'm actually defining a *module*, and some of you might want to nitpick other details in the definition. For the second group, my excuse is that the difference between a *component* and a *module* is very blurry (try googling it; everyone seems to have his own understanding). For others, comment section awaits!

### What goes inside a component?
Let's focus on the middle part of my definition - cohesive responsibilities. I'd describe this as ==likelihood of changes happening to two classes together along with their conceptual cohesion==. To provide a quick example, we can imagine that `Customer` and `MailSender` classes won't have too much in common in a typical project, so they won't be in the same component. `Mail` and `MailSender` on the other hand seem to complement each other - every new piece of information `Mail` class will hold, `MailSender` will have to reflect in the actual email sent. Therefore `Mail` and `MailSender` are good candidates to be in the same component.

Does that mean imaginary `MailController` and `MailRepository` will end up in the "mail component" too? There are consequences, so it depends. In my recent project, I decided to keep controllers inside components and so far I haven't experienced any issues. There's a great text by Simon Brown that gives some insight into his understanding of [Layers, hexagons, features and components](http://www.codingthearchitecture.com/2016/04/25/layers_hexagons_features_and_components.html). I definitely recommend reading this one, before you make up your mind for good. I also recommend to experiment on your own, because projects vary and so best solutions do.

### Creating a component
Having identified two classes that are more cohesive to each other than to the rest, we should put them together - we want component structure to be explicit, not hidden in the forest of other things, not imaginary in the mind of an architect. We have multiple means of doing that - packaging, Maven modules, Java 9 modules, OSGi etc. In most projects, this implies that we should seek packaging (or "modularization") by functionality, rather than by object type (see also: [Package Structure](http://tidyjava.com/package-structure/)).

### Encapsulating a component
Oh, I could write some cool stuff about component encapsulation, but it so happens that I already did. Make sure to check out [Java Encapsulation for Adults](http://tidyjava.com/java-encapsulation-for-adults/), if you haven't already. TLDR; ==Use expressive packaging and/or visibility modifiers and interfaces to encapsulate your component's internals (and save your ass on the dance party).==

### Reusing a component
To reuse a component, we just include it in our project (if necessary) and use it's public API. It's so simple and yet we rarely hear about great systems built of reusable components. More often we hear about the dreaded "BIG BALL OF MUD". Why?

I believe it's not (yet) mathematically proven, but component's reusability is inversely proportional to amount of magic it uses and dependencies it requires. Since poor dependency management and magic seems to be ubiquitous in software, reusability tends to suffer. We'll work through this theorem using an example. Let's take a look at the [git component from my blogging platform project](https://github.com/tidyjava/blogging-platform/tree/reusable-components-article/src/main/java/com/tidyjava/bp/git).

I claim that this component keeps track of a configured Git repository by exposing an endpoint to notify it about changes and allows it's clients to read from the repository. What can we say about this component's reusability?

Firstly, it's reusable inside this project, as long as we need to keep track of only one repository. If there were more repositories to track, we would need to change both service's and component's implementation.

Secondly, it might be reusable in other projects using Spring, but we would have to do a few things:

* deploy it as a an artifact
* bundle it with `ExceptionUtils` class that it's using, which might involve copying the class or setting a dependency on utils artifact
* make sure the other application is using a compatible version of Spring - example of how a dependency can lower reusability

Thirdly, it might be reusable in other projects that are not using Spring, but we would have to:

* deploy it with necessary dependencies (same as first 2 criteria above)
* remove Spring dependency (remove the controller, remove Spring's annotations from the service, expose lifecycle methods through component's interface) - example of how unwanted dependency and magic lifecycle can lower reusability

Lastly, in this example the only in-project dependency is a utility class, which is fine. Imagine what would happen if this component had a dependency on a more project-specific class e.g. `Post`. Anybody who'd like to use this component would have to declare a dependency on blogging-specifc stuff or reimplement non-configuration logic in the blogging platform. Therefore, ==watch for project-specific dependencies in technical components like this example one!==

Enough. I think that 2 things should be really clear about the example component:

* current implementation is not very reusable
* it has a lot of potential for reusability improvements

So..

> HAHAHAHAHA! GRZEGORZ, YOU FAILED! [TIDY JAVA](http://tidyjava.com)? BS! NON-REUSABLE JAVA! GO AND REWRITE, HAHAHA!

No, no, no. YAGNI boy, I'd rather KISS a component than make it maximally reusable "just in case". Look at how much complexity could changes mentioned above introduce. You don't want that complexity, until absolutely necessary. Therefore, the key takeaway of this part should be:

==Keep your components **potentially reusable**, so that's it's **possible** to reuse a component either directly or with a small change. Don't shape your code around unnecessary reusability.==

### Summary
By defining a *component* as a class or bunch of classes with cohesive responsibilities, we can move the term from the world of tales and legends to reality. We grab conceptually cohesive classes that might change for the same reasons and put them together into a package, module or whatever else we're using. A component should be well encapsulated to encourage loose coupling. Component's reusability is inversely proportional to amount of magic it uses and dependencies it requires. We should aknowledge this truth, but not use it as a reason to actively fight every reusability flaw until we really need it - good software design rules still apply. Happy applying!

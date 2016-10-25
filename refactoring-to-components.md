---
id: ghost-34
title: Refactoring to Components
author: Grzegorz Ziemonski
summary: Imagine you're assigned to work with an old codebase without a reasonable component structure or any structure at all. A big ball of mud! What now?! How do you get from there to nice, reusable components you'd enjoy working with?
date: 2016-10-26
tags:
    - java
    - components
    - blogging-platform
---
Imagine you're assigned to work with an old codebase without a reasonable component structure or any structure at all. A big ball of mud! What now?! How do you get from there to nice, reusable components you'd enjoy working with?

### The Ball Of Mud
For the purpose of this article, the ball of mud will be my [Blogging Platform](https://github.com/tidyjava/blogging-platform) as left after [part 4](http://tidyjava.com/solving-small-problems/). All the classes were squeezed into one package despite their very different responsibilities. Here's how it looked from the IDE perspective:

![](/img/before.png)

### Inferring the Components
In case there's no structure at all or it doesn't make any sense, a reasonable first step is to seek implicit structure in the code and make it explicit. Unless the codebase is a single God class, there must be some way to group the classes together, even if the resulting structure is suboptimal. If there is an imperfect, but workable structure already, this step can be skipped.

In the given example, it's clear that we have 3 classes directly related to posts, 1 class mostly related to markdown, 2 classes mostly related to git, 1 utility class and 1 "main" class. `MarkdownPostFactory` and `GitPostReader` look like implementations of non-existent `PostReader` and `PostFactory` interfaces, at least judging by the name. As said before, we don't care that the inferred structure is suboptimal, we just need something to work with. Let's make it explicit:

![](/img/inferred.png)

### Recognizing Suboptimality
Given a component structure, we can start seeking potential improvements. As components are just bunches of classes cooperating together for a common purpose, all standard design qualities apply to them - high cohesion, low coupling, SOLID etc. We can also use linguistic techniques to locate flaws in our componentization. Here's one:

> Name each component, it's responsibility and collaborators. Seek awkwardness and responsibility leaks.

Let's apply it to our inferred blogging platform component structure:

* Post component is responsible for presenting posts provided by an implementation of `PostReader`
* Git component is responsible for cloning and updating a given repository, and for reading posts from the repository and converting them to `Post` objects using an implementation of `PostFactory`, for the purpose of Post component
* Markdown component is responsible for creating `Post` objects from markdown files for the purpose of Post component

To me this is all just weird. Post component doesn't do anything in terms of reading and creating posts - everything is abstracted away using interfaces. Git component does a whole lot of git-related work and post-related work. It also uses Markdown component indirectly by a post-related interface. Markdown component has a very narrow responsibility and consists of only 1 class.

### Optimizing the Structure
Knowing what's exactly wrong with our components, we should be able to improve on these things. We'll start by describing the target picture and then gradually move towards it. This implies a series of refactorings, so better get your tests ready to confirm that you're not breaking anything!

Considering the weak points mentioned above, I've chosen the following target structure:

* Post component will be responsible for reading and presenting posts located in repository maintained by Git component. Converting markdown files to `Post` objects will be an internal detail of the Post component, enclosed within a `PostFactory` object.
* Git component will clone and update (on-demand) a given git repository.

After implementing the changes in the codebase, we get a structure like this:

![](/img/after.png)

### Encapsulating the Components
After slicing out proper components, it's good to encapsulate their internals using visibility modifiers and interfaces. I've written a whole article on encapsulation, which you can (even should!) check out [here](http://tidyjava.com/java-encapsulation-for-adults/).

As you can see on the image above, I have already encapsulated Git component's behaviour using `GitSupport` interface. This way, all the complexity of `GitSupportImpl` is hidden behind something as succint and innocent as:

```java
public interface GitSupport {
    File getWorkTree();
}
```

Isn't that beautiful?

### Summary
A Ball of Mud is not the end of the world. You also don't need to rewrite everything to microservices to achieve a good componentization and independent developability of features. Start by analyzing the current structure and inferring implicit componentization left to you by your precedessors. Use good design rules and other techniques to recognize suboptimality in the inferred structure. Improve on recognized flaws by setting a target picture and gradually refactoring towards it. Once the components are sliced, encapsulate them so that none of the implementation details or responsibilities leaks out ever again. Enjoy!

*This post was inspired by Tim's article with the same name. You can check it out [here](https://codingtim.github.io/refactor-to-components/).*
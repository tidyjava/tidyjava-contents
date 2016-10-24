---
id: ghost-19
title: Starting a Project - Blogging Platform #1
author: Grzegorz Ziemonski
summary: It's been some time since I started blogging and I see a lot of room for improvement. I was looking for a way to deliver more value with my posts, especially in terms of good examples. It's not possible to write a big enough example application for every single post and I can't use any of my work projects for obvious reasons. Taking all this into account, I decided to create something open source that will produce real value and act as an example in my posts. Here it comes, yet another blogging platform!
date: 2016-07-07
tags:
    - java
    - blogging-platform
---
It's been some time since I started blogging and I see a lot of room for improvement. I was looking for a way to deliver more value with my posts, especially in terms of good examples. It's not possible to write a big enough example application for every single post and I can't use any of my work projects for obvious reasons. Taking all this into account, I decided to create something open source that will produce real value and act as an example in my posts. Here it comes, yet another blogging platform!

### But why?

There are a few things I dislike about Ghost that I'm currently using:

1. It's expensive. Not that I don't have enough money, I'm just greedy. I could host it by myself, but then I have to care about updates and stuff by myself.
2. There's no good review mechanism, without giving someone almost full rights to my blog. I really need that one.
3. I'm lazy and I don't want to learn how to make it look/work better, especially that it's written in node.
4. It requires me to be online to write a blog post (not necessarily, but the preview helps a lot).
5. I didn't develop that.

I also don't want to move to other blogging platforms, because they share most of the disadvantages mentioned above, plus most of them are written in PHP.

### Overview
I want something as simple as possible. Here are a few key ideas:

* small amount of code with least dependencies (ye, I know that these two seem contradictory, I'll have to find the right balance)
* blog contents are stored in Git, preferably in a separate repo than the blog server - people can review my posts and suggest improvements via Pull Requests (they can even propose their own posts, yay!)
* blog posts are written in markdown or something as-easy
* modifying the looks is super-simple
* I have an option to run my blog locally with a live preview
* RSS feed is generated
* Comments are available via disqus

### Tools
I'm currently thinking about using the following tools to create the project:

* Java 8 - obvious choice for a Java related blog, though I might experiment a bit with Java 9 later
* Gradle as a build tool, probably the newest version - I'm not good at it, but I hate pom.xml's, so it's time to learn something else
* Spock for all kinds of tests - it will give the project some groove :)
* IntelliJ IDEA with default formatting settings - it's my favorite IDE and I don't want to bother contributors with stupid rules
* [GitHub repository](https://github.com/tidyjava/blogging-platform)

Anything else that joins the toolkit is yet to be decided.

### Small design upfront
Before I actually start coding anything, I want to share with you my general design idea, that might (not) be later implemented. I have two possible sources of contents - filesystem or a git repository, used in local and hosted mode respectively. I scan through the contents and generate HTML for each post. Over content source I set a "watch", checking for changes regularly e.g. every second. If a change occurs, it triggers re-generating post's HTML. For the views, I currently need just index with headers and a single post view. HTTP routes can be resolved dynamically - "/" to index, everything else as a possible post.

![](/img/blog.png)

To be honest, I didn't spend a lot of time preparing it. I just wanted to have a clue what I'll be building, not necessarily a complete picture.

### Final words
Here we go. You are more than welcome to share your thoughts on the project in comments and contribute on [GitHub](https://github.com/tidyjava/blogging-platform).
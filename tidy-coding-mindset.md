---
title: Tidy Coding Mindset
author: Grzegorz Ziemonski
summary: This is one of the posts that I wanted to write for a long time. This is also one of the posts that, once written, I should read myself at least once a month. It's about the proper approach to coding - the mindset that you need to have to produce high quality, well designed, **simple** software solutions. Here it comes!
date: 9999-12-31
---
This is one of the posts that I wanted to write for a long time. This is also one of the posts that, once written, I
should read myself at least once a month. It's about the proper approach to coding - the mindset that you need to have
to produce high quality, well designed, **simple** software solutions. Here it comes!

# Do The Simplest Thing...
That Could Possibly Work. This is an old, yet not so popular, principle that every developer should know and obey.
Whatever you have to do, implement the simplest, most straightforward, least generic solution that will solve the problem
at hand. Also, by "simple" I don't mean easy. 

<blockquote class="twitter-tweet" data-lang="pl"><p lang="en" dir="ltr">Easy vs. Simple <a href="https://t.co/b6r0cTD6WO">pic.twitter.com/b6r0cTD6WO</a></p>&mdash; Dave Cheney (@davecheney) <a href="https://twitter.com/davecheney/status/733225955624263680">19 maja 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

More on this principle on the fantastic [c2 wiki](http://c2.com/xp/DoTheSimplestThingThatCouldPossiblyWork.html).

# Don't Fight Dragons
Dragons don't exist, really. It's all in your head. Whenever you get a feeling that a line of code will keep growing
and growing in the future, and eventually become unstoppable, take over your codebase, your company, your family and
your life... take a break. Seriously, go for a walk, take a deep breath, play a game of table soccer. As for the line of
code... let it grow. Remember, ==code for now, not for the future==. You're not a mad dragon slayer, fighting imaginary
dragons on every corner. You're a sneaky hunter, hiding behind a tree, ready to release an arrow in the exact moment
it's needed.

```java
public void updateContactDetails(String phoneNumber) {
    ...
}
```

Everything's fine.

```java
public void updateContactDetails(String phoneNumber, String mobileNumber) {
    ...
}
```

Stay calm, keep waiting..

```java
public void updateContactDetails(String phoneNumber, String mobileNumber, String email) {
    ...
}
```

Now, kill it!

```java
public void updateContactDetails(ContactDetails contactDetails) {
    ...
}
```

Bam! You see? Patience! What if the third argument never came? Then we'd never need to refactor!

# Make Small Steps
However far you want to get in the end, your next step should be as small as possible. A single failing test, a single
line of code to pass the currently failing test, a single refactoring to perform. Whenever you face a complex problem,
you have to step back and figure out the small, easy steps to solve it. If you try to chew too much at once, you might
choke!

I have tried writing about this already [here]() and [here](). There's also a great post on small steps by Erik Dietrich
[here](http://www.daedtech.com/solve-small-problems/).

### Don't Overthink
It might happen that despite your attempts to move in small steps, you have tried to make a leap and it didn't go so well.
In this case, it might be tempting to start thoroughly analyzing the problem, find out the best solution and become
a hero. Unfortunately, more often it means mind overloading, unnecessary context switching and a huge productivity loss.
Once you recognize yourself overthinking, make sure you get back to small steps and simple solutions.

### Improve Incrementally
Have you just discovered a new concept? Read an interesting book? Learned a new pattern? You might feel like everything
you've been doing so far was wrong and you need to turn it around to save the world. This is the moment when you're
actually dangerous to the project. This state is the exact opposite of not-overthinking and making small steps. You
can turn it around to your advantage, by splitting all the improvements into smaller chunks and applying them one-by-one.
Your ideas will be validated quickly and hopefully do more good than harm.

# Keep Learning
I barely scratched the surface of the topic. Actually, I listed only the things that are particularly important to me.
There's much more to learn and discover. Read books and blogs, watch videos, go to conferences. Whatever you do,
there'll always be something to learn or some skill to improve. Keep it up!
---
id: ghost-29
title: Small Things Matter
author: Grzegorz Ziemonski
summary: Hey boy, are you doing microservices? Sure you are. Everybody does. Have you moved to 4-tier architecture already? Are your applications cloud-native? Have you containero-dockerized yourself? All of it?
date: 2016-09-03
---
Then, I bet you also have strong DevOps culture. Of course, none of these could be achieved without Management 3000.0 and Modern Agile. Oh boy, you're so awesome! But wait, wait. We're not done yet. You're using Marathon or Kubernetes, right? And Big Data, you have to do Big Data! What about your APIs? Are they RESTful enough? What about automation? You have everything autmated, right? Aaaagh, and you have to go functional! Or Reactive! Or both! REFUNCTIONAL! And don't forget about Internet of Things, ok? I'm still waiting for the glass that will display to me how full it is over the internet! You know, the world is moving so fast. Software is eating up the world! New technologies are popping up every day! We need to constantly learn new things to keep up to date! It's so hard to be a developer these days.

![](/img/rainbow_puke.jpg)

Let's call a spade a spade. That's all bullshit! Yes, don't be afraid of the word.

# BULLSHIT.

If you're a developer, your job is most likely taking problems and solving them using a bunch of classes and/or functions. `for`s and `if`s.

What does all of the above mean in this context? Nothing. Even the architecture doesn't mean so much. You pick one and follow it, bam, done.

But you know that you're missing something. That magic thing, that will prevent your code from getting overly complex, your project from becoming an unmaintanable mess.

Ok, so what causes the mess? What causes the complexity? Why so many projects fall into legacy stage?

* dependencies
* lack of tests
* long classes
* long methods
* bad names
* leaky abstractions
* broken encapsulation
* ...

# SMALL THINGS.

* Should I name this class/function/variable A or B?
* Where should this function go?
* Should I use this or that pattern? Parhaps none?
* Have I covered all cases with my tests?
* Is this piece of code readable?
* Does it fit conceptually?
* Does it represent the domain correctly?
* How may it evolve in the future?

Now, these are real problems. That's what matters. That's what decides if your project will succeed or fail. That's what you should pay most attention to. Don't forget that. Good luck!
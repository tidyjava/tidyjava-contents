---
title: Let it go!
author: Grzegorz Ziemonski
summary: A tear appears in my eye. My beauty! My child! I've just spent 3 fantastic, productive pomodoros developing this feature! Maybe I'll just leave it. The end users will have a choice! More features! Yes! But.. I'm a programmer. YAGNI, KISS... I can't just leave this complexity in there, just in case someone prefers double deployment, just to be able to review posts via pull request. My heartbeat goes crazy. And then it woke up...
date: 2016-11-01
---
I sat today morning in front of my computer, opened [KanbanFlow](https://kanbanflow.com) and started planning out my day.
First thing on the list? Enable previewing the posts, so that my wife or my friend Darek can review my latest text and suggest
improvements. I was waiting for this moment since the very beginning of the [blogging platform](https://github.com/tidyjava/blogging-platform)
project. Now, finally, I'll add the possibility to make pull requests with newly added posts for easier reviews, for better future, for better
world!

I felt excitement when clicking the "Start" button on the pomodoro timer, just about to start the implementation.
I opened Notepad++ and quickly sketched the steps for upcoming solution:

* parametrize Git integration using `git.branch` property, so that I can deploy an instance showing the newly prepared
post
* add a feature toggle for not-showing posts with future dates (By default, the platform hides future posts until the
specified date. This way you don't have to remember to manually push the post on selected publish date)
* fork the tidyjava repo and configure the fork to use `develop` branch
* deploy the forked repo

I'm ready to go. Positive energy is flowing through my body, to my fingers, hitting the keyboard in all the right places.
Neat, cool solution begins to emerge. Testing the newly added features was a bit of a challenge, but this only raised my
excitement with the upcoming feature. A little bit of genius, two pomodoros and the implementation is ready to serve!

I can't wait till the five minute break between pomodoros is over. It's not even a break, I'm sitting there ready to
deploy my newest beauty. I fork the repository, update the configuration, push everyting to GitHub. I open Heroku,
create a new application, link it to the new repository and hit deploy. It works, it works!!

Then I started realizing, that the great thing that I just built was not really necessary. Instead of deploying an extra
instance of the platform to use `develop` branch with my blog's contents, I could just push everything to `master` with
future post dates. As the platform filters out future posts, they won't be visible on the home page, but can still be
accessed by direct link. It boils down to a trade: post review via pull requests vs. complexity of extra configuration
and extra deployment.

A tear appears in my eye. My beauty! My child! I've just spent 3 fantastic, productive pomodoros developing this
feature! Maybe I'll just leave it. The end users will have a choice! More features! Yes! But.. I'm a programmer. YAGNI,
KISS... I can't just leave this complexity in there, just in case someone prefers double deployment, just to be able to
review posts via pull request. My heartbeat goes crazy. And then it woke up...

The killer instinct: You fool! You're not your code! This is not your child! This is a mistake, everybody makes mistakes.
You've just written a needless piece of code, now you delete this crap! It looks good? So what?! No emotions, just do it!
Kill it, murder, bury and forget!

And this is what you, my readers, should do with any piece of needless code. Delete it. Let it go.

<iframe width="560" height="315" src="https://www.youtube.com/embed/L0MK7qz13bU" frameborder="0" allowfullscreen></iframe>

The code never bothered me anyway.
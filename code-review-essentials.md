---
title: Code Review Essentials
author: Agnieszka Paszek
summary: Let's just say, hypothetically, that you have just started working for a new company. Finally a perfect job you have always dreamt about... or at least what it seemed to be until you haven't taken a first look at THE CODE. That's when first crisis comes along...
date: 9999-12-31
---
Let's just say, hypothetically, that you have just started working for a new company. Finally a perfect job you have always dreamt about... or at least what it seemed to be until you haven't taken a first look at THE CODE. That's when first crisis comes along... But after you gave it a deep though you realise you can face it. You're not giving up so easily. After all, it's YOU who will be working there and you CAN (and should) make it better (and I mean here much, much better).
You have to come up with a plan. So what can you do to both improve how things are right now and gain some knowledge about the project, which for certain will be useful in the further steps of your brilliant plan? The obvious answer comes to your mind - code reviews!
Just think about it - it means that no more code that doesn't meet good coding standards will be created*. You of course still have to deal somehow with the code that already exists, but at least it won't keep getting worse.**

# What are code reviews NOT about?
Some people perceive code reviews only as a way of controlling junior devs by more senior ones - making sure that they don't make any mistake, don't break anything. Actually, a comprehensive suite of tests is a solution to that problem. Reviewer should of course give some thought to code correctness, but mainly by looking at tests and checking if there are covering every aspect of a story being implemented.

# So what they ARE about?
* Attention to quality - when you write code with a belief that somebody is going to read and criticize it, then you are putting more effort. And even if you won't write it perfectly (which most probably you won't, cause nobody does that), then the reviewer can suggest you how to write it better. Maybe he knows some construct you haven't heard about, maybe some neat solution which will allow you to write less code.
* Improved code readability – when you write code, it’s easy for you to tell what it does, but when somebody else looks at it and has like a million of questions, then it means you have to improve it. You should keep making it clearer until that person will be able to understand it.
* Knowledge sharing - both programming in general and project-related. Especially when there is a new team member, who is still kind of lost in a project, others can give him hints how different modules are supposed to be used and prevent using hack solutions.
* Collective ownership - there won't be a single line of code about which only person knows what it does. Developers will have wider view of a project, which may help them design better solutions. They will know not only the code they have written. That allows more flexibility in assigning tasks – no one will a have to stick to one topic for the whole time. So they decrease the [“bus factor”](http://5whys.com/blog/category/first-steps-during-chaos).
* Common style - they help you to get a feeling of your team's coding style and to establish code conventions (it's always good to keep your code consistent).

# How should code reviews be done?
Here's a few tips which will help you to get the most out of code reviews:
* Everybody should be doing them and having them done. If you are in a small team maybe
everybody can review every line of code. In a bigger one, you can think of some rotational system.
* Code reviews should be done quickly. Don't make somebody wait a long time for your feedback. The faster the better, but it would be probably good enough to do them once a day.
* Ideally code should be reviewed after all tests were run and  tools like e.g. Sonar were used in order not to make somebody write a comments like: “you have unused import here”. And of course this should be made automatically after a pull request was created. But that requires some infrastracture in place as well as a good suite of tests. Which on the other hand requires testable code. So in some cases there might still be a long way to go, but the most important thing is to go in the right direction, even if you're making very small steps.

# Summary
Code reviews can be a really valuable technique that reaches far beyond code correctness. Performed correctly, they’ll move you towards better code quality and a collective ownership. Basic code reviews guidelines include: everyone participates in the process, they should be done as soon as possible and they doesn’t serve as a replacement for static analysis.

For more interesting information about code reviews have a look at:
[https://www.atlassian.com/agile/code-reviews](https://www.atlassian.com/agile/code-reviews)


*Or at least merged  
**Actually it may not be ideal either, cause every new feature is highly coupled with really badly designed company framework, but still...
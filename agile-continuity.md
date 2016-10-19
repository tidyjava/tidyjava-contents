---
id: ghost-24
title: Agile Continuity
author: Grzegorz Ziemonski
summary: I've already expressed [my concerns about meetings](http://tidyjava.com/very-important-meetings/), but I think there's more to say in terms of agile processes. Putting these things into practice is essential to being a good software developer. I've worked in Scrum and Scrum-like environments for a while and I have a feeling that we lack something - **continuity**.
date: 2016-08-07
tags:
    - agile
---
### Continous Communication

Not just ceremonies and other meetings. In my ideal world, these shouldn't be *needed* at all. Super-quick sync, no more than that. Anything that didn't pop up less than half an hour before the meeting, you could have communicated already. You could have discussed it with someone. With enough luck, you could have solved the issue by now.

Starting a new task? Let others know. Having a problem? Ask someone for help. Need some information? Go and ask. Cross-office issues? Make a phone call, chat using instant messenger. Send an email as the last resort. *Take action.*

### Continous Planning:

Learning about the domain, gathering requirements, getting feedback about already released features - all of these are continous processes. For me, it implies that we should continously *take action* in response to them.

You learn about a new feature to be added? Create a story. Priorities change? Move backlog items around. A defect is found? Add it to the sprint.

### Continous "Retrospective"

This should be obvious by now. Once you realize there's an issue or something you could do better, it should be communicated, discussed and some *action taken*. Obviously, some things may slip around, so a retrospective meeting could be still conducted the usual way. Ideally, the more continous things are, the less points will be brought up.

### Continous Delivery

You expected that, didn't you? I've met so many people "doing agile" without continous delivery in place. Stand-ups, plannings, retros, demos - everything is there, but delivering software. The feedback loop remains long, different processes like QA stuff are moved to just-before-the-release and things are not too agile at all. Once again, I'd like to encourage you to *take action*.

**Too short, unofficial guide to Continous Delivery**  
If you're lucky enough, you're not working in environment with legal or security constraints that actually disallow you to do anything in terms of delivery. Let's suppose so. Now, there's a huge chance that either one of the following is true:

* delivery process is already automated, because it's a long, repetetive task
* delivery process is not yet automated, because it's so simple

In case it's not automated, you have to add some automation, which depending on complexity can vary from very easy to very hard. Having this, you have to move the responsibility of deploying the application to your team. This can be done in two ways:

* talking to the right people and making it official
* breaking into the system and going YOLO

I hope the first option works for you.

Of course, I skipped some important points like automated tests, but you're smart and you know it already.

### Continous Transparency

The more continous your processes become, the more opaque your work can become to stakeholders, architects, managers and other people who actually care about what you do. Treat this as necessary evil. In the end, without them you probably wouldn't be working for the company at all. If they need something, *take necessary action* to provide it.

This is the point that causes a lot of unnecessary meetings. New features? Delivery continously and/or organize quick demos. Application's architecture? Draw diagrams on the fly or generate using tools like Structurizr. API integration details? Try Swagger and/or Spring Rest Docs. Product roadmap? (Hopefully) Look at the backlog and summarize it according to current priorities. I won't give more examples, because YMMV anyway.

### Road to Continuity

That vision of things continously evolving and moving forward seems a little bit Utopian to me. We won't and probably shouldn't eliminate standard ceremonies from the process. But making them a quick, lightweight nodding that everything's clear is something appealing and entirely possible. We can strive for it, by *continously taking action*. Every action is an atomic (or quantum) event that fills blank spots of our daily work. The more action we take, the less blank spots there are - the more continous we get. To be continued :)
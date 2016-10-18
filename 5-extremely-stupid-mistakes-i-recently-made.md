---
id: ghost-31
title: 5 Extremely Stupid Mistakes I Recently Made
author: Grzegorz Ziemonski
summary: [Michael Tharrington](https://dzone.com/users/2687707/michael-tharrington.html) from DZone crew recently suggested to me that I could write about mistakes that I make. Well, the moment couldn't be better - my team just went live with a critical application and everything that could go wrong.. went wrong! Of course, none of the solutions was written solely by me - some of it was created when we were pair programming and all of it went through rigorous code review process, which makes it even more ridiculous.
date: 2016-09-24
---
### Background
We were developing a simple webapp exposing REST API that communicates with an external provider and persists some of the results, so that we don't do too many calls, because we have to pay for each of them. The application is mostly relying on Spring Boot web and data capabilities. Two instances of the application are deployed behind a load balancer on company's servers.

### Mistake 1 - Insulting Concurrency Lords
The call to external provider takes a long time, which our client applications are not willing to wait. Because of this, we agreed to do background processing - the first time we're called we return a `DONT_KNOW_YET_COME_BACK_LATER` message[^1], the next we return a persisted result. Of course, a question "what if we get two same calls in a short enough time?" frame popped up - we have to somehow protect ourselves from calling the external provider twice with same data. Here's what I did:

```java
private Set<Long> currentlyProcessedIds = ConcurrentHashMap.newKeySet();

// further down in the code:
if (currentlyProcessedIds.add(id) {
    doTheJob(id);
    currentlyProcessedIds.remove(id);
}
```

Can you spot what's wrong with this piece of code? Take your time if my stupidity isn't visible to you at hand.

Of course, the whole thing breaks if `doTheJob` throws an exception! Dear Concurrency Lords, such an obvious violation, 4 pair of eyes looking at this and nobody noticed how we may block an id from being correctly processed ever again. Here's more concurrently-correct version:

```java
if (currentlyProcessedIds.add(id) {
    try {
        doTheJob(id);
    } finally {
        currentlyProcessedIds.remove(id);
    }
}
```

### Mistake 2 - Load was Balanced, Solution was NOT
If you're smart enough, you have noticed what's wrong already. I intentionally wrote "concurrently-correct" above, because it's still far from correct. We have two independent instances of the application behind a load balancer. This means that the problem is not only concurrent. It's also distributed!

We haven't implemented a distributed solution yet, but we obviously need to synchronize the currently processed set between the instances [^2].

### Mistake 3 - Morning Coffee Performance
The idea of background processing came from my observation that the old solution relied mostly on database performance and even in worst cases wasn't as slow as the external call. At start we suggested the guys responsible for the client application to just timeout on the first (long) call and just try again later. But this wouldn't be enough when the whole system is under heavy load. I took responsibility for measuring how big is the time window for our application to respond. I found all database queries in the old solution and ran them one by one against our databases. I collected the results, made some calculations and produced a nice table on company's wiki.

What could go wrong with that? Well, me running queries by hand, while sipping the morning coffee[^3] obviously produced other performance results than all systems running under heavy load. This meant my calculations were overly optimistic and the first thing we saw after go-live was a wall of timeouts on the client application side.

### Mistake 4 - Off by Three Orders of Magnitude
System load was only one of the reasons we overestimated our performance. The second one is even more embarassing. All testing between our application and the clients was done on a limited data set - around 1000 records. The results were promising, no warning signs.

One little detail that we somehow omitted was that just before the go-live we were supposed to fill the database with data migrated from the old solution - a million records! Increasing the data set by three orders of magnitude revealed that we forgot to set up a crucial database index - that's something you'd rather do before you go live with a critical application!

### Mistake 5 - Metrics Ain't Statistics
Business people asked us to prepare a report about requests that we make to the external provider. Since we make a lot of the calls and these are expensive, we want to give the business a way to calculate expected costs and validate the invoice that we get from the provider. Of course, business guys ain't into SQL or so - they asked for an excel report sent via email. Oh mama, us generating an excel file and sending emails? It's 2016. Look, we have this pretty board that we use for metrics. We'll add there a counter of requests for you and you have what you need!

Since we had those initial problems mentioned above, we issued some hotfixes and released the application again. After redeployment, the dashboard with counters went (seemingly) mad. What happened to the number of requests? It turned out that metrics are kept on application side and sent at regular intervals to the metric service. This means that each time we restart the application, the counter value is gone. Holy crap!

Luckily, we have means of providing correct information about the requests without the nice counter on the dashboard, but we're still to implement a real counter to keep statistics for the business.

### Summary

Just before the go-live, I told my colleagues that I'm worried, because since the development began till the end of testing phase we had no major issues. Everything went so smoothly, no big impediments on our way. It turned out I was right, something wrong has to happen! While mistakes I mentioned in this post seem obvious when I talk about them now, they weren't obvious when we were developing and testing things in hurry. It was a bit embarassing to write about it, so I hope at least you had some fun and won't make the mistakes we did!

[^1]: Actually it's "NEW_CUSTOMER", but I didn't want to confuse you with details.
[^2]: Some part of me wants to give up on this safety mechanism at all, but you know.. safety first!
[^3]: I don't actually drink coffee, just wanted to make it funnier.
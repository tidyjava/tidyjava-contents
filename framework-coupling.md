---
id: ghost-5
title: Framework Coupling
author: Grzegorz Ziemonski
summary: **Are you using frameworks the right way? Are you using frameworks for business purposes or the other way around? Are your business classes dependent on frameworks? Can I press delete button on your Spring, Guice, Hibernate, JPA dependencies and still be able to test and use your business features? If not, you might have a huge problem - high framework coupling.**
date: 2016-03-31
tags:
    - architecture
---
**Are you using frameworks the right way? Are you using frameworks for business purposes or the other way around? Are your business classes dependent on frameworks? Can I press delete button on your Spring, Guice, Hibernate, JPA dependencies and still be able to test and use your business features? If not, you might have a huge problem - high framework coupling.**

### Why is it so important?
Firstly, frameworks and libraries age much quicker than our software does. We want to upgrade them as often as possible and change them as easily as possible. In 3 years from now, current style of writing Spring applications might be totally obsolete. Or we might want to shift to another cool framework out there. Remember all those EJB applications people wrote in the past? Most of these still exist, someone has to maintain and develop them. Worse, probably right now there are many people rewritting those in rage to Spring or Jooby. And they're probably making the same mistake.
Secondly, and this is something even more harmful, high framework coupling leads to untestability. Each extra framework you couple to, you have to take into account when testing, making testing process more complex, often a lot longer and, in extreme cases impossible.

### Microservices don't change anything
One of the arguments I heard against considering framework coupling a problem is that we write microservices. That doesn't change anything. No matter what high-level architecture you choose, you will probably write similar amount of code to cover all the business features. What's the difference between having 1 huge poorly designed application and having 20 small poorly designed applications? Each of these is maybe more managable, but you have to test, release and monitor 20 instead of 1.
Then I was told that each of these applications is so small that it's easy to rewrite it. Wrong way to go. If I have a dozen of microservices, each taking 3 months to write and I want to rewrite each every 3 years, I'm getting into an endless loop of rewritting! And no, it probably won't go faster when "just rewritting", because in 3 years it won't be the same team any more - they will have to learn things from scratch or from framework-coupled code.

### Background
I first came across this problem in Uncle Bob's post [Screaming Architecture](https://blog.8thlight.com/uncle-bob/2011/09/30/Screaming-Architecture.html). I looked around my projects and saw a bunch of Spring "services", repositories and configuration classes. They all scream Spring. Then I started thinking it out and realized it's more than just class names. This is the thing that made me lose a lot of nerves and hours in the past - when I wanted to upgrade framework's version, delete one completely from the system, test something in a framework-coupled application or read code written years ago in company's internal frameworks.

### Solution
The solution is easy, but [uncomfortable](https://blog.8thlight.com/uncle-bob/2011/11/22/Clean-Architecture.html) for most developers. You have to change the way you write your applications.
Set clear boundaries for your business components. Make sure none of these boundaries are violated by framework dependencies. There is a great video about architecture and boundaries on Clean Coders: [Architecture, Use Cases, and High Level Design](https://cleancoders.com/episode/clean-code-episode-7/show).
Whenever you want to use some of the framework capabilities you should invert the dependency ([Dependency Inversion Principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle)). Sometimes it may require some extra classes e.g. interfaces or adapters, but it's really worth it. This also applies to JPA and other persistence mechanisms. You should [never use persistence data structures like business objects](https://sites.google.com/site/unclebobconsultingllc/active-record-vs-objects) - data structures and objects are different by definition!
Once you have all of the dependencies inverted you might want to put business related code into separate physical component i.e. separate JAR file. Physical boundaries are an ultimate way of protection against unwanted dependencies - your code won't compile unless you pull them in.
What should seem natural at this point, you should strive to keep your test suite as framework-independent as possible. And I don't mean frameworks like JUnit or Cucumber here. I mean that your business code should be 100% testable without using any of the frameworks your non-business components use, like Spring or Hibernate.
Another, more general rule that should be followed is: ==Keep your frameworks easy to use and easy to remove.== Even if you probably won't change all the frameworks in your application, it is more than sure that you will want to upgrade their version. Make it pleasure of using newer technologies, not pain of dealing with the old ones.

### Real life example
I worked on an orchestration system called Service Engine at an airline solutions company. The folks have built an internal framework for orchestrating services flow using Spring, BPMN engine and internal DB configuration library (also based on Spring). You could define orchestration rules in a BPMN file and see a nice diagram how the whole process works. Then, in each step you could inject meaningful data from previous steps using annotations. Lastly, you could get any configuration key from the database by simply adding an abstract method to configuration class and annotating it. Cool, isn't it?
Not at all. If you wanted to utilize any of framework capabilities you had to be dependent on all of it. This had huge and costly implications.
Firstly, they did not invert any dependencies and decided to keep everything in the same codebase. Enormous codebase, whose builds broke every day on CI servers and even more often in our IDEs.
Secondly, the system was completely untestable. How to run an acceptance/integration test in such a framework pile without running all of it? How to quickly run the whole machinery for the purpose of each test? How to satisfy all the dependencies, how to mock all relevant classes if you don't even know which are these? So they took a way around - testing from outside. That was a total killer. They can't run acceptance/integration tests using conventional CI mechanisms. They can't mock out anything. If a downline service is down, the whole testing is blocked - you can't change a single line of code and test it properly (and those services were down pretty often!). When I left the company, estimated time of single regression test phase was around 2 weeks. Madness!

![](/img/giphy.gif)

### Conclusion
Software architecture is not about cool stuff. Keep your business away from frameworks. Make the frameworks serve your application's purpose, not the other way. Code in your language of choice, not frameworks. Look around your application, try to figure out how to make it more framework-independent. Continously make sure any framework or library does not impediment your testing.

*Let me know in comments what you think about this and how's your project's situation.*
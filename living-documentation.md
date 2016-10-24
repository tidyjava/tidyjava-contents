---
id: ghost-7
title: Living Documentation
author: Grzegorz Ziemonski
summary: Tidy guide on how to create a Living Documentation in your (not only) Java projects.
date: 2016-04-14
tags:
    - documentation
---
*Tidy guide on how to create a Living Documentation in your (not only) Java projects.*

# Background

Every project needs a good documentation. Lots of people (customers, managers, QAs, new team members, even other teams) often need a way to catch up with how the system works, all of them at different levels of abstraction. This means, we need to write a lot of documentation to get everyone satisfied. But there are problems with writting so much documentation.
Firstly, nobody wants to do it, because it's boring. It can cause a lot of frustration, which leads to a decrease in productivity.
Secondly, documentation often quickly gets out of date - much like code comments do. It takes a lot of effort and discipline to keep everything up to date. Once we have a lot of it, it might even get impossible to find all relevant places to update.
Last but not least, writing it can take a lot of time we could use for coding. We don't want to be writing documentation books nobody will ever read.

**tl;dr Documentation is useful, but writing it sucks.**

# Code is your documentation

We can avoid the problems above by following a simple rule: ==keep all documentation in the codebase and make sure it's executable==.
Firstly, we can build a test suite that presents system functionality at different levels of abstraction. For business people and management, we can write acceptance tests they can read and understand. For other programmers, we write unit tests that specify concrete class' behaviour.
Secondly, we can use auto-generated docs e.g. JavaDoc, Swagger, to provide documentation for other teams, that have integrate with our application.
Lastly, we eliminate operational documentation by automating as much as possible and providing a simple readme file with relevant operational commands.
This way, writing all documentation becomes a part of our day-to-day efforts and the docs should never get out of date. This is what we call a **Living Documentation**.

# Acceptance tests
Instead of writting feature documentation, we can use acceptance tests. These tests are high-level descriptions of business requirements. They assure that you write the right code. Each test is a single scenario for implemented features. These tests are usually written in a format specific for the tool we're using.

In the perfect world, these tests should be written by business people and be a part of every user story. In the real world, programmers have to write these tests and consult them with the business.

### Tips
The important thing about acceptance tests is that they should be written and accepted before you start implementation. This means that the first thing to do, before you start working on a new story, you have to make sure these tests are in place and approved. You could also put having acceptance tests into your [Definition of Ready](https://www.scruminc.com/definition-of-ready/).

Acceptance test steps should be implemented as soon as the required components are present. Ideally, this should happen before you finish the implementation, so that you get the red-green confirmation when you're done.

### Fizz buzz example
Imagine we're writting a [Fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) application for a lazy teacher. In our acceptance tests, we will have a single scenario: we input a range and the application outputs the game sequence.

In [Cucumber](https://cucumber.io/) this scenario could be written like this:

    Feature: Fizz Buzz
  
    Scenario Outline: Game elements for a range
      Given a range <from> - <to>
      When I ask for game sequence
      Then <sequence> is presented
      Examples:
        | from | to | sequence            |
        | 1    | 5  | 1, 2, fizz, 4, buzz |
        | 14   | 16 | 14, fizzbuzz, 16    |

If you're unfamiliar with Cucumber JVM, I recommend reading about it [here](https://cucumber.io/docs/reference/jvm#java). There are a lot of other tools for acceptance testing in Java e.g. [JBehave](http://jbehave.org/) or [FitNesse](http://www.fitnesse.org/).

# Unit tests
Unit tests are a specific kind of documentation. They show developers how to use a certain unit (i.e. class) and what results can you expect. Reading a good unit test can be more valuable then reading the class itself - it's much easier to tell what the class really does when you see the results, not just the algorithm.

### TDD
If you want the documentation (and test) to be complete, you want to cover every single line of code you write. For this purpose, [Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) can be very helpful. If you're, by any chance, not familiar with TDD, I strongly recommend you learning and using it.

### Fizz buzz unit tests
Covering the whole TDD process, even for such simple application, would require a separate post and I suppose everyone knows how to write a bunch of simple unit tests. However, for educational and entertainment purposes, I recommend watching the excellent Fizz buzz kata video [here](https://vimeo.com/65902219).

### Parametrized tests
To make our unit test documentation even more expressive, we can merge multiple tests that share a common concept into parametrized tests with more descriptive (or at least more readable) names. This gives you a better overview of unit's functionalities and looks much cleaner. The test is less like a bunch of asserts and more like a specification.

Writting parametrized tests is possible in JUnit, but I feel like the Spock/Groovy syntax is better for the purpose. Parametrized (in Spock: data-driven) tests for generating a single Fizz buzz element could look like this:

    class FizzBuzzSpec extends Specification {
        def fizzBuzz = new FizzBuzz();
    
        def 'string value for number not divisible by 3 or 5'(input, stringValue) {
            expect:
            fizzBuzz.forNumber(input) == stringValue
    
            where:
            input | stringValue
            1     | "1"
            2     | "2"
        }
    
        def 'fizz for number divisible by 3 but not 5'(input, _) {
            expect:
            fizzBuzz.forNumber(input) == "fizz"
    
            where:
            input | _
            3     | _
            6     | _
        }
    
        def 'buzz for number divisible by 5 but not 3'(input, _) {
            expect:
            fizzBuzz.forNumber(input) == "buzz"
    
            where:
            input | _
            5     | _
            10    | _
        }
    
        def 'fizzbuzz for number divisible by 3 and 5'(input, _) {
            expect:
            fizzBuzz.forNumber(input) == "fizzbuzz"
    
            where:
            input | _
            15    | _
            30    | _
        }
    }

We could even put all cases in one test parametrized by input and output, but then we would lose useful information contained in test names.

In this case we don't get much advantage from parametrization, because we have only one argument that affects the output. In more complex examples, this approach can give a significant improvement in test readability. If you're unfamiliar with [Spock](https://github.com/spockframework/spock), I recommend you reading it's [documentation](http://spockframework.github.io/spock/docs/1.0/index.html) and writing a few tests yourself.

Even though I like the tests to end up in parametrized form, in the implementation process I still write a separate test method for each case and only merge them once the unit is complete.

# API docs
If you're providing an API for other programmers, make sure you have an easy way to auto-generate it. The last thing you want to do is manually codifying the API in some sort a wiki page. For APIs distrubuted as JAR files, old good JavaDoc should be enough. If the API is distributed as a REST service, you could use something like [Swagger](http://swagger.io/). In general, any tool is fine as long as the docs are generated automatically.

# Operations
In case of operations, we want automate as much as possible and document as little as possible. In dev environments build and local deployment should be possible from the IDE using simple configurations. In other environments, the whole delivery process should be automated using a CI server like [Jenkins](https://jenkins.io/) and/or an automation tool like [Puppet](https://puppet.com/) or [Ansible](https://www.ansible.com/). Whole tooling configuration should reside in the codebase - we don't want to configure anything by hand. Instructions on using the tools should be kept in a short readme file.

# Conclusion
To achieve Living Documentation, it has to be executable and reside in the codebase. High-level documentation for the business can be prepared in the form of readable acceptance tests e.g. in Cucumber. Low-level documentation for the programmers should have the form of unit tests, preferably written using TDD and converted into data-driven specifications later. API docs for other teams should be auto-generated using tools like JavaDoc or Swagger. Operations related documentation should be replaced by automation and a single, short readme file how to use it.
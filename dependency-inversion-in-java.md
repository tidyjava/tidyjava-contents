---
id: ghost-6
title: Dependency Inversion in Java
author: Grzegorz Ziemonski
summary: Begineer guide to using the power of Dependency Inversion with examples in Java.
date: 2016-04-07
tags:
    - java
---
*Begineer guide to using the power of Dependency Inversion with examples in Java.*

### Introduction
[Dependency Inversion](https://en.wikipedia.org/wiki/Dependency_inversion_principle) allows us to make low-level details depend on high-level policies, opposing the flow of control. Whenever a high-level policy changes, the low-level details must adapt. However, when the low-level details change, the high-level policies remain untouched. This is a key concept in OO and one of the main sources of it's power.

In this post we will cover three important techniques to achieve dependency inversion - dependency injection, abstraction and adapter pattern. We will also see how these three work together to solve a real problem.

### Problem
We write an application that aggregates weather information from multiple sources i.e. APIs. Our initial implementation might look like this:

    package com.tidyjava.weather;

    import com.tidyjava.weather.api1.WeatherApi1;
    import com.tidyjava.weather.api2.WeatherApi2;

    public class WeatherAggregator {
        private WeatherApi1 weatherApi1 = new WeatherApi1();
        private WeatherApi2 weatherApi2 = new WeatherApi2();

        public double getTemperature() {
            return (weatherApi1.getTemperatureCelcius() + toCelcius(weatherApi2.getTemperatureFahrenheit())) / 2;
        }

        private double toCelcius(double temperatureFahrenheit) {
            return (temperatureFahrenheit - 32) / 1.8f;
        }
    }

This solution has serious design flaws. Firstly, our weather aggregator knows about concrete APIs it uses e.g. it has to deal with the fact that WeatherApi2 returns temperature in Fahrenheits. This means that our high-level aggregating policy is dependent on details of low-level data sources. Just look at the imports, that's scary stuff!
![](/img/weather_initial.png)
Secondly, our aggregator creates concrete objects by itself. It both rises the level of coupling and violates [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).
Thirdly, each new API requires us to change WeatherAggregator class. This is an [Open-Closed Principle](https://en.wikipedia.org/wiki/Open/closed_principle) violation.

### Dependency Injection
[Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) is a pattern that moves the responsibility of dependencies creation outside of the object that is using them. Dependencies are passed into the object using a constructor or a bunch of setter methods. Dependency Injection can be used to enable Dependency Inversion.
#### Solving the creation problem
We will use Dependency Injection to enable Dependency Inversion in our code. Here's a simple constructor solution:

    public class WeatherAggregator {
        private WeatherApi1 weatherApi1;
        private WeatherApi2 weatherApi2;

        public WeatherAggregator(WeatherApi1 weatherApi1, WeatherApi2 weatherApi2) {
            this.weatherApi1 = weatherApi1;
            this.weatherApi2 = weatherApi2;
        }
        // rest of the logic
    }

This will force concrete API objects to be created outside of our aggregator and passed in during it's creation. Please note how simple Dependency Injection can be - it doesn't require sophisticated frameworks like Spring or Guice.

### Abstraction
Abstraction is the key to Dependency Inversion. We put it between the high-level and low-level classes. High-level classes use the abstraction, while the low-level classes implement them.
![](/img/dependency_inversion-1.png)
It's important to understand that the abstraction is logically coupled to the high-level class, despite the physical coupling of inheritance. Abstraction is modelled so that it serves high-level needs, which isn't necessarily "comfortable" for low-level classes to implement. Since the flow of control still goes from high-level to low-level classes, we say that dependency opposes the flow.
![](/img/flow.png)
==This concept is very simple, but extremaly important for effective OO design. Take your time to fully understand what *inversion* means in this context.==
#### Inverting the dependency
We are now ready to invert the dependency. We can do it by creating a common abstraction for our Weather APIs. It could be an interface like this:

    package com.tidyjava.weather;

    public interface WeatherSource {
        float getTemperatureCelcius();
    }

Then, we have to make our Weather APIs implement the interface. WeatherApi2 would look like this:

    package com.tidyjava.weather.api2;

    import com.tidyjava.weather.WeatherSource;

    public class WeatherApi2 implements WeatherSource {
        @Override
        public double getTemperatureCelcius() {
            return toCelcius(getTemperatureFahrenheit());
        }

        private double getTemperatureFahrenheit() {
            // some logic
        }

        private double toCelcius(double temperatureFahrenheit) {
            return (temperatureFahrenheit - 32) / 1.8f;
        }
    }

Finally, we can fix our aggregator class by making it use WeatherSource:

    package com.tidyjava.weather;

    import java.util.List;

    public class WeatherAggregator {
        private List<WeatherSource> weatherSources;

        public WeatherAggregator(List<WeatherSource> weatherSources) {
            this.weatherSources = weatherSources;
        }

        public double getTemperature() {
            return weatherSources
                .stream()
                .mapToDouble(WeatherSource::getTemperatureCelcius)
                .average()
                .getAsDouble();
        }
    }

Take a look at the imports again - concrete APIs are gone. Now it's the WeatherSource interface that all classes are pointing towards. WeatherAggregator is dependent on an abstraction that fits it's needs and all the APIs have to implement it. It's the low-level API classes that are dependent on the high-level policy of returning temperature. Do you feel the power?
![](/img/weather--1-.png)
Also, we can now add as many APIs as we wish and WeatherAggregator will remain untouched. Pure awesomeness!

### Adapter Pattern
[Adapter Pattern](https://en.wikipedia.org/wiki/Adapter_pattern) is a design pattern that allows us to use a class via an interface it does not derive from. It is implemented in two variations - class adapter (using inheritance) and object adapter (using composition). In our example, we will use the object variation. Make sure to check te link above, if you are not familiar with the pattern.

Adapter pattern can be used to invert a dependency on a class that is not under our control e.g. comes from an external library. In such case, we make the adapter derive from our abstraction and call the desired class.

#### Spring Weather API
Imagine that SpringWeatherApi comes from an external library named Spring Weather and provides the temperature in Fahrenheits.

    public class SpringWeatherApi {
        public double getTemperatureFahrenheit() {
            // some logic
        }
    }

We want to use it in our aggregator, but we can't make it implement WeatherSource directly. What we can do, is create an adapter that implements WeatherSource and internally uses SpringWeatherApi. Here's the code:

    public class SpringWeatherApiAdapter implements WeatherSource {
        private SpringWeatherApi weatherApi;

        public SpringWeatherApiAdapter(SpringWeatherApi weatherApi) {
            this.weatherApi = weatherApi;
        }

        @Override
        public double getTemperatureCelcius() {
            return toCelcius(weatherApi.getTemperatureFahrenheit());
        }

        private double toCelcius(double temperatureFahrenheit) {
            return (temperatureFahrenheit - 32) / 1.8f;
        }
    }

Look what we did here. By using simple composition we protected our WeatherAggregator from being dependent on Spring Weather. You can use this technique to protect yourself from being dependent on any framework or library class. Now, [it's the library that serves you and your architecture, not the other way around](http://tidyjava.com/framework-coupling/). That's a cool thing!

### Bonus: Composite
By making a small change to WeatherAggregator, we could make it implement WeatherSource. This would give us a [composite pattern](https://en.wikipedia.org/wiki/Composite_pattern) - WeatherSource clients wouldn't be able to tell if they are getting data from one source or twenty.

    public class WeatherAggregator implements WeatherSource {
        private List<WeatherSource> weatherSources;

        public WeatherAggregator(List<WeatherSource> weatherSources) {
            this.weatherSources = weatherSources;
        }

        @Override
        public double getTemperatureCelcius() {
            return weatherSources
                .stream()
                .mapToDouble(WeatherSource::getTemperatureCelcius)
                .average()
                .getAsDouble();
        }
    }

### Summary
Dependency Inversion is a powerful tool for making our software more flexible. Whenever we see that our high-level code depends on low-level details, we should invert the dependency using abstractions. If the low-level code is already released (e.g. in a library), we can use the Adapter Pattern to connect it with the abstraction. Dependency Inversion often comes together with Dependency Injection - the latter facilitates the former.
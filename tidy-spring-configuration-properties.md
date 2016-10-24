---
id: ghost-9
title: Tidy Spring - Configuration Properties
author: Grzegorz Ziemonski
summary: Spring has become extremely popular among Java developers. After all, it's a great project with tons of useful features. I decided to share with you a list of practices I use to work effectively on Spring applications, avoid unnecessary [framework coupling](http://tidyjava.com/framework-coupling/) and achieve a tidy application architecture. The whole set of practices was too long for one article so I divided it into smaller parts. In this one, I will focus on effective handling of **configuration properties**.
date: 2016-04-28
tags:
    - java
    - spring
---
Spring has become extremely popular among Java developers. After all, it's a great project with tons of useful features. I decided to share with you a list of practices I use to work effectively on Spring applications, avoid unnecessary [framework coupling](http://tidyjava.com/framework-coupling/) and achieve a tidy application architecture. The whole set of practices was too long for one article so I divided it into smaller parts. In this one, I will focus on effective handling of **configuration properties**.

# YAML
It is possible to use [YAML](https://en.wikipedia.org/wiki/YAML) files as a source of properties. I find this format more readable than conventional properties file, especially when you get used to it. It also supports collections and other data structures.

Example .properties file (from [Spring Boot docs](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-yaml)):

    environments.dev.url=http://dev.bar.com
    environments.dev.name=Developer Setup
    environments.prod.url=http://foo.bar.com
    environments.prod.name=My Cool App

And the YAML equivalent:

    environments:
        dev:
            url: http://dev.bar.com
            name: Developer Setup
        prod:
            url: http://foo.bar.com
            name: My Cool App

Of course, if your set of properties is pretty flat and simple, you can stick to classic properties files. But remember, complexity grows over time and later you might be reluctant to deal with it. If you already have some complex property configuration, YAML is definitely the way to go.

# Avoid spreading @Value's
In annotation-heavy Spring applications, I often see `@Value` annotations spread all over the place. This seems like a really bad idea to me - it makes each of the classes dependent on a property name (or even multiple property names!). If you want to change it, you have to update all files referencing it. If you want to find out where the property is used, you have to go for a text search. Don't do this to yourself.

If you follow the tip (from [previous post](http://tidyjava.com/tidy-spring-starting-a-project/)) to extract business features into Spring-independent components, then the ability to spread @Value around is already limited (good for you).

# @ConfigurationProperties
A much better approach to handling properties is to use a few configuration classes annotated with `@ConfigurationProperties`. If you follow the same naming in config file and the class, you don't even have to explicitly specify the key's name.

For the example set of properties:

    externalApi:
        url: http://externalApi.com
        port: 8080
        username: foo
        password: bar

The configuration class would look like this:

    @Component
    @ConfigurationProperties(prefix = "externalApi")
    public class ExternalApiConfiguration {
        private String url;
        private String port;
        private String username;
        private String password;

        // getters and setters
    }

With this approach, when you want to find/change property handling, you just look for a single class with a descriptive name.

It's very useful to establish a naming convention for these classes e.g. `SomethingConfiguration` or `SomethingProperties`. Once you have a few of these in a single component, you might go for a common package e.g. `config`.

# Custom structures
In case of complex configurations e.g. lists or nested key-values, we can write custom structures to hold the properties in a better manner. All we have to do is create a class with desired properties and a bunch of getters/setters.

For the environments example from Spring docs:

    environments:
        dev:
            url: http://dev.bar.com
            name: Developer Setup
        prod:
            url: http://foo.bar.com
            name: My Cool App

The configuration class could look like this:

    @Component
    @ConfigurationProperties(prefix = "environments")
    public class EnvironmentsConfiguration {
        private Environment dev;
        private Environment prod;
    
        // getters and setters
    
        public static class Environment {
            private String url;
            private String name;
    
            // getters and setters
        }
    }

Custom configuration structures reduce some boilerplate code and enhance readability. You avoid writing lots of similar methods like:

    public String getDevUrl() { ... }
    public String getDevName() { ... }
    public String getProdUrl() { ... }
    public String getProdName() { ... }

# Validation
When we use `@ConfigurationProperties`, we get out-of-the-box support for properties validation. We should make use of that, so that our application fails fast at start instead of misbehaving.

The `ExternalApiConfiguration` mentioned above, with validation, could look like this:

    @Component
    @ConfigurationProperties(prefix = "externalApi")
    public class ExternalApiConfiguration {
        
        @URL
        private String url;
        
        @Max(value = 65535)
        private int port;
        
        @NotBlank
        private String username;
        
        @NotBlank
        private String password;

If you can't find a predefined validator for your needs, it's very easy to write your own. All you have to do is create an annotation and associate it with a `ConstraintValidator`.


    @Constraint(validatedBy = MyConstraintValidator.class)
    @Target({ElementType.FIELD})
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MyConstraint {
        String message() default "Default message used when constraint fails";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};
    }

And the associated validator:

    public class MyConstraintValidator implements ConstraintValidator<MyConstraint, String> {

        @Override
        public void initialize(MyConstraint myConstraint) {}
    
        @Override
        public boolean isValid(String value, ConstraintValidatorContext c) {
            return value.equals("Tidy Java");
        }
    }

It is also possible to specify an error message to display if validation fails. Unfortunately, in current Spring Boot version, if validation fails, Spring prints a long, ugly exception message and adding your own does not help. Therefore, I would recommend to avoid cluttering the code with custom messages.

In my projects, I rarely have a property without any validation. It's also not uncommon for me to write a custom validator, since it's both useful and easy. If you haven't tried it before, give it a shot.

# Business properties
It often happens that we need access to configuration in our business classes. In such cases, we should use [Dependency Inversion](http://tidyjava.com/dependency-inversion-in-java/) to avoid framework coupling. We create an interface on the business side, then implement it in the "main" component and inject the configuration into appropriate objects.

Suppose we wanted to use environment configuration in the business part of the application. We could create an interface like this:

    public interface ExternalApiConfiguration {
        String getUrl();
    
        int getPort();
    
        String getUsername();
    
        String getPassword();
    }

And then make the other class implement the interface:

    @Component
    @ConfigurationProperties(prefix = "externalApi")
    public class SpringExternalApiConfiguration implements ExternalApiConfiguration {

        @URL
        private String url;

        @Max(value = 65535)
        private int port;

        @NotBlank
        private String username;

        @NotBlank
        private String password;

       // getters and setters
    }

These interfaces don't get a separate package. Instead, they "sit" next to the classes using them. This way we express the logical biding between the interface and the using class.

# Segregation
Another important thing is to avoid putting all properties into a single configuration class. Instead, go for a few specialized ones. Configuration tends to stack up over time and we might end up with something too big and unmanageable. The same applies to configuration interfaces. Too much knowledge can finally become a problem (see: [Interface Segregation Principle](https://en.wikipedia.org/wiki/Interface_segregation_principle)).

# Conclusion
YAML is a viable replacement for classic properties files. To avoid configuration mess, we should favor a few classes with `@ConfigurationProperties` over `@Value` spread all over the place. Complex property structures can be enclosed in separate classes. To avoid misbehaviour caused by configuration, we can use built-in validation with annotations to fail fast. Configuration that has to be accessed from the business part should be abstracted using well-segregated interfaces.

### Other parts of the series:
[Tidy Spring - Starting a Project](http://tidyjava.com/tidy-spring-starting-a-project/)  
[Tidy Spring - Services](http://tidyjava.com/tidy-spring-services/)
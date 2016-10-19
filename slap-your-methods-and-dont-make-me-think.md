---
id: ghost-25
title: SLAP Your Methods and Don't Make Me Think!
author: Grzegorz Ziemonski
summary: In my [recent post](http://tidyjava.com/blogging-platform-in-java-3/) about creating a blogging platform I posted a piece of code like this:
date: 2016-08-12
tags:
    - java
---
```java
    public MarkdownPost(Resource resource) {
        try {
            this.parsedResource = parse(resource);
            this.metadata = extractMetadata(parsedResource);
            this.url = "/" + resource.getFilename().replace(EXTENSION, "");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

Never mind the smart constructor (which hasn't caused any problems so far!), there's an important rule violated - **Single Level of Abstraction Principle**.

The principle says that every statement in a method should be on the same level of abstraction. If we try to read this method top to bottom we get something like this:

* parsed resource is initialized by the result of parsing the resource (yay!)
* metadata is initialized by extracting metadata from the parsed resource
* url is initialized by... STRING CONCATENATION WITH A REPLACE USING A CONSTANT AND A MAGIC EMPTY STRING

That's not good. **A method should read smoothly.** Let's fix it:

```java
    public MarkdownPost(Resource resource) {
        try {
            this.parsedResource = parse(resource);
            this.metadata = extractMetadata(parsedResource);
            this.url = urlFor(resource);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private String urlFor(Resource resource) {
        return "/" + resource.getFilename().replace(EXTENSION, "");
    }
```

Not sure if `urlFor` is the best name, but at least the original method looks and reads much better. But we're not done with levels of abstraction yet.

When reading solutions for [Sam Atkinson's code challenges](https://dzone.com/users/1338295/samberic.html), I noticed that in most of them there's a missing level of abstraction! Most solutions for [chemical challenge](https://dzone.com/articles/java-code-challenge-chemical-symbol-naming-part-on) have some kind of `isValid` method, that does `String` operations all way long.

You may ask what's wrong with this - it's like 10 lines of code. I'll tell you what - it's 10 lines of *decoding*. I don't want to do that. I don't want to sit and think *what the hell did you mean*. I want to read a method and know what's happening in there right away, what was the intention, what are the business rules behind it!

This is an adaptation of **Don't Make Me Think** rule to coding. In case of methods, it can often be achieved by introducing another level of abstraction. Let's look at another example.

You want to validate a newly submitted user account. The criteria is simple:

* username is unique
* password contains special characters
* email is in correct format
* user is/claims to be adult

Let's write a method for it:

```java
    public boolean isValid(User user) {
        ...
    }
```

What do you expect inside `isValid`? A database call to check uniqueness? A regexp check for the email? Date calculations? Hopefully none of these.

```java
    public boolean isValid(User user) {
        return isUnique(user.username)
            && containsSpecialCharacters(user.password)
            && hasCorrectFormat(user.email)
            && isAdult(user.birthdate);
    }
```

Of course, the database call and other checks will have to be done just below this method. But if someone wants to check how a user is validated, the first thing he sees are clearly defined rules, all on the **same level of abstraction**. He **won't have to think**.

Now it's your turn to do the thinking and **SLAP** your methods! Despite my unwillingness to think, I still have to correct mine too ;)
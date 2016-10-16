---
title: Java Encapsulation For Adults
author: Grzegorz Ziemonski
summary: For proper understanding of this article I'd like you to close your eyes and imagine something. But since having closed eyes and reading at the same time is not an option, we'll skip the "close your eyes" part and jump straight into "imagine".
date: 2016-10-08
tags:
    - java
---
### Intro

Imagine you're going to a big dance party with your crush. You're really into him/her, so you don't want to mess up. You want to be sexy, charming and funny. Now, **WHAT COULD POSSIBLY GO WRONG?**

For the sake of this article you and your crush are objects and your life is the component you're in. I'll show you why you need encapsulation at various levels of abstraction to succeed.

### Method level
So you come to the party and notice that nobody's dancing yet, everybody's shy, staying by the tables. Of course, you don't want to lean out, so you offer your crush to eat something. Imagine his/her reaction if you asked like this:

> Maybe we should go to the tables, take some food, put it in our mouths, chew for a bit, yummy, yummy and then swallow?

Or imagine everybody's dancing already and you want to hit the dancefloor, so you say:

> Maybe we should go in the middle of those people and start roaming with our legs interleaving to the rythm of music?

You'd speak like an idiot!

Methods are the way your objects speak and act. When you name a method by enumerating the things it does, you make your object speak like an idiot.

```java
// bad
you.putInMouthAndChewAndYummyYummyAndSwallow(food);
you.roamAndInterleaveYourLegsToTheRythm(yourCrush);
```

**Therefore**

==Encapsulate methods' behavior using short, descriptive, **intention-revealing** names to make your objects speak and act normally.==

```java
//good
you.eat(food);
you.dance(yourCrush);
```

### Class level - Fields
You might know how to speak normally, but still be pretty nervous and do something stupid e.g. get drunk. Of course, you really care that things work out well with your crush, so you won't drink too much on purpose. But what if I told you that you have no control over your alcohol digestion? Any of your stupid friends around can go by and make you drunk. Seems insane?

Well, this is what happens to objects with `public` fields. By making them `public`, everyone can read or, even worse, change them!

```java
class You {
    // bad
    public Mouth mouth;
}
```
```java
you.mouth.contents.add(lotsOfAlcohol); // ugh!
```

**Therefore**

==Encapsulate your objects' fields by making them `private`. This way you can fully control access to them i.e. prevent unwanted access.==

```java
class You {
    // good
    private Mouth mouth;
}
```

### Class level - Accessors
Now imagine you have some control - people have to ask you to drink, but you still agree every time they ask. Have the situation got any better? You might ask "why do I agree every time?".

Well, auto-generated getters and setters never say no! Of course, there are *some* questions you always answer positively e.g. someone offering you a free pizza and there are *some* cases when auto-generated getter or setter is what your object needs. But in most cases, it's doing more harm than good.

```java
class You {
    private Mouth mouth;

    // bad
    public Mouth getMouth() {
        return mouth;
    }
}
```
```java
you.getMouth().getContents().add(lotsOfAlcohol); // ugh!
```

**Therefore**

==Don't auto-generate getters and setters for all of your object's fields. That's not any better than making the fields `public`.==

```java
class You {
    private Mouth mouth;

    // nothing. good!
}
```

### Class level - Solution
Okay, getting drunk is not the right way to impress your crush. You want to show what's best about you without any stupid actions. How do you keep things under control?

As written above: methods are your objects' way of speaking and acting.

**Therefore**

==Expose a few public methods with meaningful names and clear intentions that will show the very best of your object!==

```java
class You {
    private Mouth mouth;

    // good
    public void drink(Beverage beverage) {
        if (beverage.containsAlcohol()) {
            throw new NoThankYouException();
        }
        mouth.put(beverage);
    }

    public void dance(YourCrush yourCrush) {
        embrace(yourCrush);
        sway();
        yourCrush.whisper("I'm glad you're here");
    }

    ...
}
```

### Component level - Separation
You didn't speak like an idiot and you didn't get drunk. Everything goes well until you see your ex running at you and screaming. You look around and wonder how this happened? Your ex is not supposed to be at this party. Who let him/her in?! And then you see it.. There's no door! Everyone could come in and mess with your plans! WTF?!

Making all classess public and leaving no indication, which classes should and which shouldn't be used outside of the component means anyone can use any class, in any way he likes. An object like your ex can come in and mess around. This is the missing door of your component.

![](/content/images/2016/10/wrong.png)

**Therefore:**

==Encapsulate your components internals by making it explicit what is your components public API. You can achieve that by marking only selected classes as public or through expressive package structure (or both).==

![](/content/images/2016/10/better.png)

### Component level - Abstraction
Your ex didn't get into the party, but still knew that you went there and you're into someone else already. This made him/her so depressed that he/she committed suicide. Now, this is crazy! How did your ex know this stuff in the first place? Well, this is what happens when you give your ex your phone, when you meant to give just the phone number.

Making the public API of your component consist of concrete classes can be equivalent to giving your ex the phone instead of the number. Your client objects depend on your internals and can, or worse, have to adapt to changes inside your component.

**Therefore:**

==Unless you don't care about such dependencies, make the public API of your component consist of interfaces instead of concrete classes.==

![](/content/images/2016/10/best.png)

### Finale
You had the talk, you didn't get drunk and your ex didn't cause you any trouble. Your crush is another step closer to fall in love with you! Most importantly, there's no negative tension around - the relationship between you, your crush and your ex remained healthy.

Do you know how do we call a healthy relationship in programming? **LOOSE COUPLING!**
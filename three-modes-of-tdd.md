---
title: Three Modes of TDD
author: Grzegorz Ziemonski
summary: Among the discomfort of being sick over the weekend, I also had the *pleasure* to do some reading - Kent Beck's [TDD by Example](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530) in this case[^1]. This inspired me to write something about TDD - a topic that polarizes people a lot. Reading different opinions about TDD, I found that most TDD opponents don't know what it really means and haven't even partially mastered it. Here's my first attempt to stop this trend a little bit.
date: 2016-10-04
tags:
    - java
    - TDD
---
One thing that's nice about TDD is that it lets you to make progress, even if you don't have the final solution in your mind right away. Depending on your knowledge of the problem, possible solutions and their implementations, you can chose one of three modes to move forward. We'll take a closer look at this, using an oversimplified example of calculating square's area.

### Obvious Implementation
Given you know the problem, possible solutions and their implementations very well, there's no point in messing around with stupid tricks or games. The example of square's area shows that very well.

You start with a test that describes the problem:

```java
@Test
public void testArea() throws Exception {
    Square s = new Square(3.0);
    assertEquals(9.0, s.area(), 0.0);
}
```

Then you write the obvious implementation:

```java
class Square {
    private double edge;

    Square(double edge) {
        this.edge = edge;
    }

    public double area() {
        return edge * edge;
    }
}
```

Bam, done! Things work so well when the problem is simple enough (that's why I chose this one). But let's assume that you don't know how to code it up right away. What then?

###Fake It, 'Til You Make It
Given you know the problem and solutions, but the way you code them up is not immediately obvious to you, you can use a trick called *Fake it, 'til you make it*. Given the same test as above, you could start your implementation like this:

```java
class Square {

    Square(double edge) {
    }

    public double area() {
        return 9.0;
    }
}
```

See? You're "faking" that the class works just to make the test pass. The next thing to do is to realize *Why does this constant satisfy the requirements of the test?*. Relax, let the magic flow through your body straight to your brain. Oh yes, it's the square of 3! Code it up:

```java
class Square {

    Square(double edge) {
    }

    public double area() {
        return 3.0 * 3.0;
    }
}
```

Now, you're still faking, but you're closer to the final solution. You gather brain power that exceeds it's capacity, your head starts electrizing, your hair looks like on memes with Albert Einstein and you got it! It couldn't be more obvious! You just need to replace the constants with the `edge`. Here you go again:

```java
class Square {
    private double edge;

    Square(double edge) {
        this.edge = edge;
    }

    public double area() {
        return edge * edge;
    }
}
```

Great! But what if you don't even know the solution?! How do you make progress in this case?

###Triangulation
This is heavy magic. At this point, TDD opponents on DZone are no longer reading, they're already writing angry comments about how stupid this article is. But we don't stop. We want power, more power! To unleash the power of triangulation, we have to get back to the roots. What's the most dumb example of a square? A square with edge of 0![^2]

```java
@Test
public void testAreaWithZeroEdge() throws Exception {
    Square s = new Square(0.0);
    assertEquals(0.0, s.area(), 0.0);
}
```

With triangulation, we'll be doing similar steps as in *Fake it 'til you make it*. We'll start by hardcoding the result.

```java
class Square {

    Square(double edge) {
    }

    public double area() {
        return 0.0;
    }
}
```

In this case, we assume that we don't know yet the final answer, so we can't generalize based on this constant. Instead, we'll provide another test that leads us closer to the final solution - the next most dumb square:

```java
@Test
public void testAreaWithEdgeOfOne() throws Exception {
    Square s = new Square(1.0);
    assertEquals(1.0, s.area(), 0.0);
}
```

Now the old solution obviously fails. We have to do something to make it work for both cases. Edge of 0, means area of 0, while edge of 1 means area of 1. What could it be? No. This can't be true. Are you saying that area is equal to the edge?! Well, for now it is.

```java
class Square {
    private double edge;

    Square(double edge) {
        this.edge = edge;
    }

    public double area() {
        return edge;
    }
}
```

It works. Let's keep going. Edge of 2!

```java
@Test
public void testAreaWithEdgeOfTwo() throws Exception {
    Square s = new Square(2.0);
    assertEquals(4.0, s.area(), 0.0);
}
```

Holy moly, it broke again! What do you do? You think, take a walk, get a shower, drive a car, meditate and finally...

```java
class Square {
    private double edge;

    Square(double edge) {
        this.edge = edge;
    }

    public double area() {
        return edge * edge;
    }
}
```

You got it! Step by step. From nothing to `0.0`. From `0.0` to `edge`. From `edge` to `edge * edge`. It wasn't that hard. Congratulations!

###Summary
In reality you won't be solving such simple problems, but the principles remain. If you know the solution and it's implementation right away, just code it up - it's the obvious implementation way. If you know the solution, but coding it up appears complicated - *fake it 'til you make it*. Also, go for *fake it 'til you make it*, when things unexpectedly mess up (aka your tests fail). This often happens when you have algorithms with indexes or lot's of nested calculations. It's no shame to slow down and do it step by step instead of trying MAX_INT combinations of indexes or brackets. Finally, if you don't know the solution and you're exploring, try triangulation. Go step by step, from the simplest cases to the most complicated ones. From very specific, stupid implementation, to the one that solves all your requirements.

[^1]: And [The Skull Throne](https://www.amazon.com/Skull-Throne-Book-Demon-Cycle/dp/0345531493/), but I'll save this one for another post.
[^2]: I'm not even sure if that's a valid square, but for the sake of example, please assume so!
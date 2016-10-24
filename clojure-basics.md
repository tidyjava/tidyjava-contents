---
id: ghost-15
title: Clojure Basics
author: Grzegorz Ziemonski
summary: In this post, I will try to share with you some of the Clojure's (Lisp for the JVM) beauty. *Note: I'm still learning the language, so I might be oversimplifying things. If you're a seasoned Clojure programmer, you risk a heart attack reading this :)*
date: 2016-06-02
tags:
    - clojure
---
At the university, I didn't have Lisp (or any other functional language) classes. Then I started working as Java developer and didn't have much contact with functional programming. At that time when I saw a piece of code like this:

    (defn square [x]
        (* x x))
  
    (deftest square-test
        (is (= 0 (square 0)))
        (is (= 1 (square -1)))
        (is (= 1 (square 1))))

I thought that you have to be mad to use it. However, since functional programming has become more and more popular, I decided to give it a try and changed my mind (or became mad). In this post, I will try to share with you some of the Clojure's (Lisp for the JVM) beauty.

*Note: I'm still learning the language, so I might be oversimplifying things. If you're a seasoned Clojure programmer, you risk a heart attack reading this :)*

### Hello World!

    (println "Hello world!")

This is the "Hello World" program in Clojure (using REPL). Syntax is pretty basic:

* Everything goes in parens:

        (...)

* Function name goes first:

        (function-name ...)

* Then go arguments, if there are any:

        (function-name arguments)

The `square` function, defined in the first listing, would be called like this:

    (square 2)

As you can see, there aren't more parens in Clojure than there are in Java, you just move the paren to the left!

    Java:                 Clojure
    square(2);            (square 2)
    square(square(2));    (square (square 2))

### Math operations

Math operations like `+` or `*` also go first in the expression. Actually, these symbols represent Clojure functions. It might take a while to get used to it.

    (+ 2 2)
    (- 2 2)
    (* 2 2)
    (/ 2 2)

Actually, this notation isn't any harder than the "normal" one - just remember that operation goes first.

    Java:             Clojure:
    2 + 2             (+ 2 2)
    square(2);        (square 2)

### Defining a function

To define a function we use the `defn` macro.

1. We call `defn`:

        (defn ...)

* Then we declare function's name:

        (defn function-name ...)

* Then we declare the parameters in square brackets:

        (defn function-name [param] ...)

* Finally we put the body:

        (defn function-name [param1 param2]
            expression1
            expression2)

Last expression in the body acts as a return statement.

    Java:                             Clojure:
    int f() { return 1; }             (defn f [] 1)
    int g(int i) { return i; }        (defn g [i] i)
    
Now the square function should be easy to understand. We take an argument `x` and return `(* x x)`. Piece of cake!

### Conditions

There is an `if` construct (special form) in Clojure, but it works a bit different than the Java keyword. It works the same as Java's [ternary operator](https://en.wikipedia.org/wiki/%3F:) - returns one of two values based on a condition. Here's the usage:

1. We call `if`:

        (if ...)

* Then we enter a condition:

        (if condition ...)

* Then goes the value to be returned when the condition is true:

        (if condition then)

* Lastly, we can put a value to be returned when the condition is false:

        (if condition then else)

An example might help a lot in understanding:

    (if (> x 0) "positive" "zero or negative")
    (if (< x 0) "negative" "zero or positive")

If we want to chain conditions, like if-else in Java, we should use the `cond` macro. It's syntax looks like this:

1. Macro call:

        (cond ...)

* List of conditions with associated expressions:

        (cond
            (< x 0) "negative"
            (> x 0) "positive")

* Optional "else" expression:

        (cond
            (< x 0) "negative"
            (> x 0) "positive"
            :else "zero")

### Solving a real problem

Enough learning for today. It's time to save the world by tackling the very famous "Fizz buzz" problem.

We'll start by defining a function taking one argument and returning it:

    (defn fizzbuzz [x] x)

This solves problems for non-divisible numbers. Let's make a "fizz":

    (defn fizzbuzz [x]
        (if (= x 3) "fizz" x))

That's a naive solution, because it doesn't work for 6, 9 etc., but we'll solve that later. Let's add a buzz:

    (defn fizzbuzz [x]
        (cond
            (= x 3) "fizz"
            (= x 5) "buzz"
            :else x))

"Buzz" is there. With `cond` in place, adding "fizzbuzz" is trivial:

    (defn fizzbuzz [x]
        (cond
            (= x 3) "fizz"
            (= x 5) "buzz"
            (= x 15) "fizzbuzz"
            :else x))

Perfect. It's high time to make the "fizz", "buzz" and "fizzbuzz" work for other numbers than 3, 5 and 15. We'll use the `mod` function for this:

    (defn fizzbuzz [x]
        (cond
            (= 0 (mod x 15)) "fizzbuzz"
            (= 0 (mod x 3)) "fizz"
            (= 0 (mod x 5)) "buzz"
            :else x))

Notice, we had to move the "fizzbuzz" case up, because of the "fizz" and "buzz" conditions. That's it, world saved!

### Conclusion

Despite the first impression, Clojure is a pretty simple language. Main constructs like functions, math operations or conditions are pretty similar to other languages. What's important, the language is capable of solving super-complicated problems like "Fizz buzz". I hope you had some fun!

**Extra watching**:  
[Clojure for Java Programmers by Rich Hickey](https://www.youtube.com/watch?v=P76Vbsk_3J0)  
[Clojure by Uncle Bob](https://www.youtube.com/watch?v=SYeDxWKftfA)
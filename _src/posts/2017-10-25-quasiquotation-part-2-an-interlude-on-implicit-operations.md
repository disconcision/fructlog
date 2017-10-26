    Title: Quasiquotation Part 2: An interlude on implicit operations
    Date: 2017-10-25T17:18:41
    Tags: racket
    Authors: disconcision

You see implicit operations all the time in math notation, so often that it's easy to forget they're there. In fact, the fact that we can (usually) forget them is kind of the point. Encapsulation, abstraction, information hiding, etc. etc. Consider the following inoffensive term (in standard mathematical notation):

```
f(10x)
```

In words, this denotes the application of some function f to ten times the value of a variable x. Straightforward, but I assert that in this term, white space - the space between the symbols - is overloaded, not once, but thrice, to represent three different implicit operations:

1. The gap between the f and the first parenthesis represents function application (let's call this **app**). Although this is perhaps the single most frequently used operation in mathematics, its simple existence is hardly even remarked on; at least outside of a programming languages class.

2. The gap between the 1 and the 0 represents an operation which I'll call **dec**, the contructor for decimal notation. This is a left-associative binary operation which means "multiply the left argument (digit) by 10 and add it to the value of the right argument".

3. Finally, the gap between the 0 and the x represents regular multiplication.

Even though we rarely talk about these operations, especially the first two, they are essential for understanding the semantics behind the syntax. If we fully expanded the above notation, this time using Racket prefix-order s-expression syntax, we'd get the following 'desugared form':

```racket
(app f (multiply (dec 1 0) x))
```

Ugly and inconvenient, but at least it's clear exactly what we mean.

Next up: [Quasiquotation Part 3: Implicit Operations in a Lispy Context](http://disconcision.github.io/fructlog/2017/10/quasiquotation-part-3-implicit-operations-in-a-lispy-context.html)

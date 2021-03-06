    Title: Quasiquotation Part 1: Quote
    Date: 2017-10-25T15:39:05
    Tags: racket
    Authors: disconcision

Before there was **quasiquote**, there was **quote**. Quote is a fundamental part of every lispy language, including Racket. Quote allows us to concisely represent [s-expression](https://igor.io/2012/12/06/sexpr.html)-based data; in particular, source code. Specifically, it allows us to *literalize* source code by contextualizing what we *mean* when we use *symbols* and *lists*.

Our goal is to be able to represent source code *within* source code. The simplest way to represent a thing is just to present the thing itself. Unfortunately, this representation is already taken! Say we simply provide a piece of code:

```racket
(add1 (add1 1))
```

The standard semantics of *stuff* in a source file is to interpret that *stuff* as something to be executed or evaluated. In other words, the representation above typically carries with it the implicit meaning of "do this thing"; source code is usually read in the *context of evaluation*. By contrast, if we wish to inhabit a more *literal context* - to talk about the code syntax in-itself - the most traditional approach is to consider the source as a string; in other words, to "quote" it:

```racket
"(add1 (add1 1))"
```

Although almost all extant languages ultimately use strings of characters for their stored representations, this way of *literalizing* source code is pretty bare-bones. Especially in the context of lispy languages, where programs explictly tree-structured via nested lists. In Racket, lists are most prosaically represented via the [list constructor](https://docs.racket-lang.org/reference/pairs.html#%28def._%28%28quote._~23~25kernel%29._list%29%29). Let's rewrite the above string representation of the source as a list-constructor based representation:

```racket
(list "add1"
    (list "add1" "1"))
```

This is... not much of an improvement, to say the least. It is far more verbose, and it still requires string representations for the atomic elements. In the case of "1", which is a literal (bare value), it's safe just to drop the quotes. Since literals (including 1) are distinguished by the fact that they evaluate to themselves, we are reasonably safe equivocating between their evaluative and literal representations.

```racket
> (list "add1"
    (list "add1" 1))
```

 But we can't just write **add1** without the quotes; Racket typically treat 'bare' symbols as variable references. Here, **add1** is an identifier representing a unary function, and in the above literal representation we just want to *represent a reference* to that function, not have it expanded in-line. So at minimum, we need a way to distinguish symbols-as-variable-references from symbols-as literals. To this effect, we introduce a syntactic form called **quote**:

```racket
(list (quote add1)
    (list (quote add1) 1))
```

 Now, at this point it's at least clear what we mean, though at the cost of almost completely sacrificing readability. Let's try to recover it! First, we introduce the following abbreviation: **'x**, which expands to **(quote x)**:

```racket
(list 'add1
    (list 'add1 1))
```

 But we've still got those pesky list constructors. I mean, ideally, we'd just want to write:

```racket
'(add1 (add1 1))
```

But what exactly would that mean? We need to decide what it means to quote a list; that is, to make a list-literal. Luckily, it's simple to define quote in a way that makes this work. Quoting a list will imply two things: First, a bracketed structure under a quote is in fact a list: it expands to a list constructor. Second, the elements of that list are themselves quoted. For sublists, this is applied recursively. For symbols and literals, we've already shown what this means above. To be explicit, the above expression expands as follows:

```racket
'(add1 (add1 1))
(quote (add1 (add1 1)))
(list (quote add1) (quote (add1 1)))
(list (quote add1) (list (quote add1) (quote 1)))
(list (quote add1) (list (quote add1) 1)
```

And that's really all there is to quote. Now it likely seems like we've gone a long way to get almost nothing at all. We can summarize what's gone before as "if you want to refer to the code itself instead of its result under evaluation, stick a ' in front of it". This is *almost* a fair assessment, but let's take a moment to carefully consider what we've gained.

Next up: [Quasiquotation Part 2: An interlude on implicit operations](http://disconcision.github.io/fructlog/2017/10/quasiquotation-part-2-an-interlude-on-implicit-operations.html)

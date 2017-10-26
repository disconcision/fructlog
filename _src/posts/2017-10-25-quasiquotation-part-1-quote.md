    Title: Quasiquotation Part 1: Quote
    Date: 2017-10-25T15:39:05
    Tags: racket
    Authors: disconcision

Before there was Quasiquote, there was Quote. Quote is a fundamental part of every lispy language, including Racket. What does it do for us? It allows us to concisely represent s-expression-based data; in particular, source code. Specifically, it allows us to *literalize* source code, by contextualizing what we *mean* when we use *symbols* and *lists*.

What we want to do is be able to represent source code *within* source code. We need a method of representation, and the simplest way to represent a thing is just to present the thing itself. Unfortunately, this representation is already taken! Say we simply provide a piece of code:

```racket
(add1 (add1 1))
```

The standard semantic interpretation of stuff in a source file is that it's something to be executed or evaluated. In other words, the representation above typically carries with it the implicit meaning "do this thing"; we'll call this the *evaluative context*. By contrast, if we wish to inhabit a more *literal context* - to talk about the code in-itself - the most traditional approach is to consider it as a string; in other words, to "quote" it.

```racket
"(add1 (add1 1))"
```

Although almost all extant languages ultimately use strings of characters for their stored representations, this way of *literalizing* source code is pretty bare-bones. Especially in the context of lispy languages, where programs are s-expression-based; that is, they take the form of lists, which are sequences of atoms and sublists. Lists are most prosaically represented via the *list constructor*. Let's rewrite the above string representation of the source as a list-constructor based representation:

```racket
(list "add1"
    (list "add1" "1"))
```

This is... not much of an improvement, to say the least. It is far more verbose, plus is still requires string representations for the atomic elements. or the case of "1", which is a literal or bare value, it's safe just to drop the quotes, since 1 evaluates to itself; (quote some-literal) is just some-literal:

```racket
> (list "add1"
    (list "add1" 1))
```

 But we can't just write "add1" without the quotes; lisps typically treat 'bare symbols' as variable references. "add1" is a variable representing a function, and in the above representation we just want to make reference to that function, not have it expanded in-line. So at bare minimum, we need a way to distinguish symbols-as-variable-references forms symbols-as literals. To this effect, we introduce an operation called quote:

```racket
(list (quote add1)
    (list (quote add1) 1))
```

 Now, at this point it's at least clear what we mean, though at the cost of almost completely sacrificing readability. Let's try to recover it. First, we introduce the following abbreviation: 'x, which expands to (quote x):

```racket
(list 'add1
    (list 'add1 1))
```

 But we've still got those pesky list constructors. I mean, ideally, we'd just want to write:

```racket
'(add1 (add1 1))
```

But what exactly would that mean? We need to decide what it means to quote a list; that is, to make a list-literal. What we need turns out to be simple. Quoting a list implies two things: one, the the bracketed structure is in fact a list (expands to a list constructor), and two, that the elements of that list are themselves quoted. For sublists, this is applied recursively. For symbols and literals, we've already shown what this means above. To be explicit, the above expression expands as follows:

```racket
'(add1 (add1 1))
(quote (add1 (add1 1)))
(list (quote add1) (quote (add1 1)))
(list (quote add1) (list (quote add1) (quote 1)))
(list (quote add1) (list (quote add1) 1)
```

And that's really all there is to quote. Now it likely seems like we've gone a long way to get almost nothing at all. We can summarize what's gone before as "if you want to refer to the code itself instead of its result under evaluation, stick a ' in front of it". This is mostly a fair assessment, but let's more carefully examine what we've gained.

Next up: [Quasiquotation Part 2: An interlude on implicit operations](2017/10/quasiquotation-part-2-an-interlude-on-implicit-operations.html)

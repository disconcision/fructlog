    Title: Quasiquotation Part 3: Implicit Operations in a Lispy Context
    Date: 2017-10-25T17:29:04
    Tags: racket
    Authors: disconcision


In standard Racket code, symbols are most typically employed as identifiers. The first use of an identifier is to bind a value as a variable:

```racket
(define a 1)
a
> 1
```

Note that, like variable binding, variable reference is itself a syntatic form. It is used so frequently in most languages that it is almost always implicit, but if we wanted to be explicit about what we mean when we just state a variable we might instead write:

```racket
(define a 1)
(var-ref a)
> 1
```

Note that var-ref is not actually a form in Racket; it's just my own shorthand. So now, in context, the atomic case of quote is what allows us to refer to a symbol without it being implictly interpreted as a variable reference:

```racket
(quote a)
> 'a
```

Note again that although it looks like the quote form is being transformed into something, this is purely do to the fact that the Racket pretty-printer uses a syntactic sugar for quote. It's really just giving what we put in back to us. We can confirm this by using the sugar ourselves:

```racket
'a
> 'a
```

Or by checking directly:

```racket
(equal? 'a (quote a))
> #t
```

And a quoted symbol is a literal. Specifically, a symbol-literal. At the risk of repetition: Quotation literalizes its input.

The non-atomic case for quote might seem a shorthand to express list-literals. And this is certainly a worthy feature! But there is another implicit operation linking here; our humble friend, function application. Consider that there are two central uses for lists in Racket. First, there are lists of data, constructed using the list constructor:

```racket
(list 1 2 3)
> '(1 2 3)
```

Let's think carefully about why the quote is necessary. Why can't (1 2 3) itself represent a simple list of three literal elements?

```racket
(define a-list (1 2 3))
> error
```

In Racket source files, bare unquoted lists are interpreted as follows: First, a bare list is checked against a list of the base syntactic forms of the language; define, if, etc. If the bare list doesn't match any of those forms, it is expanded into a function application:

```racket
(define a-list (app 1 2 3))
```

And 1 isn't a function, so you get an error.

To be clear, I'm not arguing that having these implicit operations for commonly used forms is in any way a bad thing. It's a virtual necessity for a readable language. The default context for understanding the meaning of code, the semantics, is provided by the process of evaluation. It makes perfect sense, then, that we'd syntactically streamline the most frequently used tools of evaluation: variable reference and function application.

But it's also important that we remember that, in an evaluative context, these forms are there, lurking behind the scene. Thus the quote form is necessary in order to be able to refer to source code, or really any list-structured data, in-and-of-itself, in a form which closely tracks with its appearance in the evaluative context.

In other words, the cost of implicit evaluation forms is explicit literalization forms for symbols and lists, and the quote form is how we cheat our way out of paying this cost.

We *want* to be able to write:

```racket
(define (f x)
    (add1 (add1 x)))
```

*instead* of:

```racket
(define (f x)
    (app add1 (app add1 (var-ref x))))
```

*but* that means we have to literalize the code as:

```racket
(list
    (list (lit define) (lit x))
    (list
        (list add1)
        (list (lit add1) (lit x))))
```

*unless* we use quote as a recursive shorthand for both symbol and list literals:

```racket
(quote (define (f x) (add1 (add1 x))))
```

The lesson here is that while implicit operations enhance readability, which operations we want to make implicit depends on context. Quote allows us to open windows into a mirror world, embedding *literal interludes* into an *evaluative context*. Negotiating between the two requires that we are canny mental parsers, carefully considering the context to determine the meaning of a given symbol.


Next time we'll look at expanding our notion of quotation to abstract source code through templates.

Next up: [Quasiquotation Part 4: Unquote?](http://disconcision.github.io/fructlog/2017/10/quasiquotation-part-4-unquote.html)

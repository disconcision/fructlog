    Title: Quasiquotation Part 5: Quasiquote!
    Date: 2017-10-25T17:50:15
    Tags: racket
    Authors: disconcision

### A Review: Parsing Quote

First, let's review how to (mentally) parse code when there are (simple) quote forms in the wild.

When encountering a symbol: Let's say you see the symbol "a". Are you inside a quote form? Then you're looking at (quote a), a symbol literal. Otherwise, are you in a binding form like *define*? If so, you're at a binding site for the identifier a. Otherwise, you're looking at a(n implicit) variable reference form, **(var-ref a)**.

For lists: Let's say you see the list "(a b c)". Are you inside a quote form? If so, you're looking at (list 'a 'b 'c), a list literal. Otherwise, does the expression denote a special syntactic form (like *if*)? If not, then you're looking at a function application.

### Introducing Quasiquote

Now, we introduce a new form, quasiquote, sugared as "\`". The first and most important thing to note about quasiquote is that, if a quasiquote form doesn't contain any unquotes, it behaves exactly like quote:

```racket
(quasiquote (if a b c))
> '(if a b c)
```

```racket
`(if a b c)
> '(if a b c)
```

If we use an unquote within quasiquote, the effect is precisely what we tried to achieve before: unquoted portions within a quasiquotation are evaluated normally. BUT within a quote form, both unquote AND quasiquote are just treated as symbol literals, and have no effect. Note that this is **not** a special case for quote; quasiquote and unquote are just treated like everything else!

```racket
(define a #t)
`(if ,a ,a ,a)
> (if #t #t #t)
```

```racket
'(if ,a ,a ,a)
> '(if ,a ,a ,a)
```

```racket
'`(if ,a ,a ,a)
> '`(if ,a ,a ,a)
```

There is one additional complexity with quasiquote, which you probably don't have to worry for a while, but is worth thinking about for the sake of completeness: What happens if we use a quasiquote inside a quasiquote?

```racket
(define a 'b)
`(if `(if ,a #t #f) 1 2)
> ????
```

There are two ways we might think about implementing this. The first (**NOT THE WAY IT ACTUALLY WORKS IN RACKET**) is simply to ignore nested quasiquotes:

```racket
(define a 'b)
`(if `(if ,a #t #f) 1 2)
> '(if `(if b #t #f) 1 2)
```

After all, why would we want to deal with nested levels of quasiquotation? What would it actually even mean? However, this is NOT the implementation used in scheme/racket. The reason (I think??) is that we want to be able use quasiquotation for macros - for rewriting code - and what's even better than rewriting code? Rewriting code that rewrites code! That is, we want the ability to use quasiquoted templates which, when evaluated, themselves return quasiquoted templates.

Thus, we have the notion of *levels of quasiquotation*; a parameter we must keep track of when parsing, expressed as a non-negative integer. In the example above, the unquote containing "a" is under two levels of quasiquotation. Therefore it is not evaluated:

```racket
(define a 'b)
`(if `(if ,a #t #f) 1 2)
> '(if (if ,a #t #f) 1 2)
```

If instead we unquoted a twice, we'd get the following:

```racket
(define a 'b)
`(if `(if ,,a #t #f) 1 2)
> '(if `(if ,b #t #f) 1 2)
```

Note that we've preserved both the nested quasiquote AND one level of unquote around "b". So what we're left with could itself be used as a template for further processing.

When you're parsing code in your head, think about it like this: Regular code (not inside a quote or quasiquote form) is at *quasiquote level 0*. This means it is evaluated like.... wait for it... regular code.

If we're reading code from the outside-in, every time we encounter a quasiquote, we increment the quasiquote level by 1, and every time we encounter an unquote, we decrement by one. If the quasiquote level is greater than 0, we interpret what we see (including symbols, lists, quotes, and quasiquotes) as literals. If the level reaches 0, we evaluate normally. If it goes under 0, like if we found an unquote without any surrounding quasiquote, we get an error.

That's basically the full picture of how Racket treats quasiquotation! But like I said, for use at a single level of 'meta' you only need to worry about quasiquote levels 0 (business as usual) and 1 (literalize until unquoted).

(Side note: There is an additional form of unquote called **unquote-splicing** (sugared as **,@**), which is used to 'splice' a variable representing a list into a parent list. [You can read more about it here](https://docs.racket-lang.org/reference/quasiquote.html).)

Next up: The stunning conclusion: [Quasiquotation Part 6: Quasiquotation in the context of Racket/Match](http://disconcision.github.io/fructlog/2017/10/quasiquotation-part-6-quasiquotation-in-the-context-of-racket-match.html)

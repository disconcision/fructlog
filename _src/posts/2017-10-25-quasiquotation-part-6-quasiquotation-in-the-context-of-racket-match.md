    Title: Quasiquotation Part 6: Quasiquotation in the context of Racket/Match
    Date: 2017-10-25T18:09:23
    Tags: racket
    Authors: disconcision

#### A brief introduction to match:

```racket
(match <target>
    [<pattern> <template>] ...)
```

An individual match clause consists of a pattern and a template. The pattern is 'matched' against the template, either comparing sub-constituents literal-to-literal, or binding literals to 'wildcard' variables. The template is just regular racket code. If we use quasiquote/unquote in our template, it's going to work exactly as I've described in the previous part. One common use for pattern matching in Racket is term rewriting and reduction; that is, parsing and evaluating code in languages which we ourselves define. This very often involves 'recognizing' some pattern of code, and transforming it according to a template.

For code rewriting, we want to do two things: *destructure* nested lists representing our code according to patterns we specify, and then *restructure* those pieces according to templates we specify. Quasiquotation as described above provides a rich way to restructure; wouldn't it be nice if we could destucture our code in a way that is syntactically symmetric? That is, instead of *imperatively* specifying how to take apart and put together expressions using list operations, we  *descriptively* illustrate the transformations themselves.


####Using Quasiquotation to Destructure:

Say we wanted to rewrite an 'if' statement to swap the consequence and alternative. If we already captured the three components of our original "if" as variables a, b, and c, we'd use quasiquotation to restructure our new "if" using \`(if ,a ,c ,b).

So what we'd like to write is something like this:

```racket
(match '(if #t 1 2)'
[`(if ,a ,b ,c) `(if ,a ,c ,b)])
```

While it's not hard to intuit what's going on in the left-hand side, let's be totally explicit about what's going on. Remember, 'destructuring' is something new here; just because we want to do it using the same syntax as regular quasiquotation doesn't mean the semantics is going to be the same, although as we'll see it's very close.

First, we should be explicit about the way the match form treats regular, non-quoted data. Remember that in regular racket code, a symbol (say, a) appearing outside a quote-type form and outside a binding form is treated as a(n implicit) variable reference form **(var-ref a)**. In the left-hand side of a match template, on the other hand, a 'bare symbol' is treated as an in-line *variable binding form*, **(var-bind a)**. Remember that var-ref and var-bind are my shorthands here; they won't work within Racket. However, if you want to be explicit about binding forms in your patterns, you can use the var form. That is, both these matches are precisely equivalent:

```racket
(match '1 [a "yes"])
(match '1 [(var a) "yes"])
```

So just like how in the right-hand-side (template) of a match clause, being at quasiquote level 0 means that 'bare symbols' are considered variable reference forms, on the left-hand-side (pattern) of a match clause, being at quasiquote level 0 means bare symbols are considered variable binding forms. Similar, at non-zero quasiquote levels, symbols are in both cases considered as symbol-literals, and lists are in both cases considered as list-literals:

```racket
(match '(3 4) [`(a ,a) `(a ,a)])
> match error
(match '(a 4) [`(a ,a) `(a ,a)])
> '(a 4)'
```

In the first case, the first a is at quasiquote level 1, so it is interpreted as the symbol-literal 'a. This is not the same as the literal 3, so the match fails. The second a, contained in the unquote, is at quasiquote level, so it would be interpreted as a binding form, if it hadn't been so rudely interrupted by the previous mismatch. The second example finally gives it it's chance to shine. On the template side, the first a is at quasiquote level 1, and so is treated as a literal; the second is at level 0 and so is treated as variable reference form.

This example is nearly fully illustrative; aside from the fact that the pattern matches and binds (destructures) and the template constructs and references (restructures), the semantics of quasiquote are almost fully symmetric, in a similar sense of symmetry as between var-bind and var-ref.

####Levels of quasiquotation within match

The only real difference is that pattern-side quasiquotation doesn't care about higher levels of quasiquote. That is, we don't need to (and in fact, can't) use multiple unquotes to 'escape from' multiple quasiquotations. An unquote always resets the quasiquotation level to 0.

```racket
(match '(1 `(,2))
    [`(,a `(,,b)) `(,a ,b)])
> match syntax error
```     

I'm not sure exactly why this behavior was chosen, but it's likely because there's no obvious use case for multiple levels of nesting in a pattern, since the pattern itself is not returned; after matching succeeds or fails, all that matters is the bindings that were made. Here's my (failed) attempt to find a use for multi-level quasiquotation in patterns:

```racket
(match '(1 `(,2))
    [`(,a `(,b)) b])
> ',2
```

What if we wanted to rewrite the above to extract just the *2*, without the leading *unquote*? Why can't we just:

```racket
(match '(1 `(,2))
    [`(,a `(,,b)) b])
> error
```

But even if multiple levels of quasiquote were implemented, my attempted solution still doesn't make sense. Note that in the match's target, the unquote represents a symbol-literal, whereas in the pattern template, both unquotes represent unquote forms. It just ain't gonna match!

More to the point, we can already accomplish what we want without modifying match, we just need to be slightly clever:

```racket
(match '(1 `(,2))
    [`(,a `(,(list 'unquote b))) b])
> 2
```

And with that, we're done! If you've made it this far, you now know just about everything you need to implement quasiquotation yourself. All that stands in your way is the [Fear of Macros](http://www.greghendershott.com/fear-of-macros/all.html).

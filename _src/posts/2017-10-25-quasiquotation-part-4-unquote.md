    Title: Quasiquotation Part 4: Unquote?
    Date: 2017-10-25T17:41:22
    Tags: racket
    Authors: disconcision

First, let's review how to (mentally) parse code when there are quote forms about.

When encountering (a wild) symbol: Let's say you see the symbol "a". Are you inside a quote form? Then you're looking at (quote a), a symbol literal. Otherwise, are you in a binding form like define? If so, you're at a binding site for the identifier a. Otherwise, you're looking at a(n implicit) variable reference form, (var-ref a).

For lists: Let's say you see the list "(a b c)". Are you inside a quote form? If so, you're looking at (list 'a 'b 'c), a list literal. Otherwise, does "a" represent a special syntactic form (like *if*)? If not, then you're looking at a function application.

Simple, no? Now let's get fancy.

The ability to represent code literally, as data, is all well-and-good, but what if we want to do something with it? What if we want to build up code from smaller parts, or break it down into subcomponents?

Technically, we already have everything we need in Racket's basic list-handling functions. We can use list-reference functions like *first* and *rest* to extract components, and list-builder functions like *list*, *list\**, and *append* to build up composites. For deeply nested lists though this can start to look obscure and indirect.

To be concrete, let's consider the case of code templates. For example, the *if* form is an expression which consists of three other expressions, denoted by lower-case letters:

```racket
'(if a b c)
```

Let's say we had values for these three component expressions and wanted to use the 'abstract' quoted form above as a template to build a 'concrete' if statement:

```racket
(define bindings '((a 1) (b 2) (c 3))')
(define if-template '(if a b c))
```

Now, it wouldn't be too hard to map or iterate through our template, replacing each instance of a particular symbol literal with the corresponding value given by the bindings. But notice that what we're really doing is just manual variable substitution, something the compiler already knows how to do! So why don't we try to make it do the work for us? What would we need?

We'd need some way of telling the compiler to take some part of a quoted form 'less literally'; that is, take it as a variable reference (or function application) instead of a symbol-literal (or list-literal). Let's call this new form unquote, which has the sugared representation ",".

What we want:

```racket
(define a 1)
(define b 2)
(define c 3)
(define if-template '(if ,a ,b ,c))
if-template
> '(if 1 2 3)
```

Now, if you try this, you'll see it doesn't work. Why didn't the language designers choose to make unquote work this way? First, remember that quote is a very core part of the language. Changing it for this kind of usage might have unintended consequences elsewhere. But also note that it limits our expressiveness. We've added a new form - unquote - to our language; that is, Racket code has a new bound symbol. But code is data; if unquote is code, we'd like to be able to represent code containing unquote as a literal using the quote form. But if every time we encounter an unquote we jump out of the quote form, this just doesn't work; a quoted form will never evaluate to a form containing an unquote. That is, there is nothing we can do to produce the following *as output:*

```racket
> '(if (unquote a) (unquote b) (unquote c))
```

or more tersely:

```racket
> '(if ,a ,b ,c)'
```

The unquoted segments would be evaluated normally, producing either a value or an error. So we'll have to something *just slightly* more complicated...

Next up: [Quasiquotation Part 5: Quasiquote!](quasiquotation-part-5-quasiquote.html)

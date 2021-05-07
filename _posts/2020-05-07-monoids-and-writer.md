---
layout: post
title: Monoids and the Writer Monad
category: Category Theory
---

**These are inspired by the text _Category Theory for Programmers_ by Bartosz Milewski and the subject of category theory. These act as personal notes for my own learning
and potentially yours, the reader, but these aren't intended to be wholly comprehensive.**

A good way to learn about category theory is to think about different
categories, so here are a few.

# Monoids

## Orders: The Categories of $\le$ relations
Suppose we have a category where the morphism between objects $a$ and $b$ is
the less-than-or-equal relation. Why is this a category? We need to check
the category laws which are the following:

A category $C$ consists of

- a class of objects,
- a class of arrows or morphisms which map from a source object to target object,
- a binary operator which composes morphisms such that for morphisms $f, g$ which
map $A \to B$ and $B\to C$ respectively, we can define a morphism $h = g\circ f$ and
$h$ is in our category.

There are two laws that composition in a category must obey, there's the implicit law that I just defined,
i.e. $h = g \circ f \in hom(C)$, and there's the law of associativity

**Associativity**

For all morphisms $f, g, h$ in our category,

$$
f \circ (g \circ h) = (f\circ g) \circ h = f \circ g \circ h
$$

**Identity**

For every object $A$ in our category, there's an arrow which is called
the unit of composition, denoted $\mathbf{id}_A$. This is an arrow of an
object to itself. When composed with an arrow $f$ which goes from $A\to B$

$$
f \circ \mathbf{id}_A = f
$$

and

$$
\mathbf{id}_B \circ f = f
$$

We have defined our category and it's class of objects as well as morphisms. We simply need
to check the laws of compisition.

Do we have identity morphisms? Yes, every object $a$ will have a relation $a\le a$.

Do we have associative composition? Yes, if $a\le b$ and $b\le c$ then $a\le c$

The name of this set with this relation is called a pre-order. If we add stronger condition
such that if $a\le b$ and $b\le a$ then they are the same object, we would have a partial
order.

# Monoid
A monoid outside of category theory is known as one of the simplest type of algebraic structures you can have
in abstract algebra. It's simply a set, with an associative binary operation and the set contains an element
which behaves as the identity. This sounds familiar...

These structures are quite common and show up in a range of situations for example natural numbers under addition
form a monoid as for all $a,b,c\in\mathbb{N}$

$$
a + (b + c) = (a + b) + c
$$

and we have identity 0 such that

$$
0 + a = a = a + 0
$$

For computer scientists such as myself, the first time I came across monoids was through Haskell. In Haskell,
monoids are a well known type-class

```haskell
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
```

Relating it to our previous mathematical definition, `mempty` corresponds to our identity element and
`mappend` is our associative binary operator. However, in Haskell we can't explicitly define the properties
of these functions, and we leave that to the programmer (proof assistants to the rescue). We could declare
our natural set of natural numbers as a monoid in Haskell as the following

```haskell
class Monoid Int where
    mempty = 0
    mappend = (+)
```

### The Monoid Category
When we think of the monoid category, we come to an interesting more abstract understanding of the structure
of a monoid. Let's shift perspective and consider the binary operator of monoid to be a morphism which shifts
things about the set we act on - we'll call these adders to avoid confusion with traditional functions.
Relating this to the function signature of `mappend:: m -> m -> m`, via currying, our adders have this signature
`m -> (m -> m)` i.e. they take some m and return a morphism from `m` to `m`.
This makes sense when we consider that `mappend` always generates an element
within our set already. Our category will also have the adder which takes a morphism and doesn't "move" it, i.e. an identity. This can be thought of as the the result of `mappend` when we pass `mempty` as a
function parameter.  Morphisms are these adder functions which map to this object. Trivially we have an identity and composition laws are obeyed by the definition of morphisms. In fact our category
consists of only one object. With this we have our monoid category which is simply the category with one object and appropriately behaved morphisms.

# Kleisli and the Writer Category
Pure functions in functional programming are functions which have no side effects, this means that they don't modify or access some global state and they return the same output for every input. We may wish to allow for side-effects when we code to allow for operations that rely on them e.g. I/O operations.
We can model side effects using category theory.
Programmers who may have experience with FP languages such as Haskell will be familiar with monads which are used to model stateful computations.
Monads are a type of category, but I won't discuss those here just yet. Instead I will start with a discussion of how would we model a function that logs and traces it's execution.
The key issue with this problem is as we perform functions, we want to add some extra functionality ontop by embellishing the return types of these functions. In our example, we would want to produce some type of log of information derived
from the return type of a function. We define a `Writer` category through Haskel.

```haskell
type Writer a = (a, String)
```

our morphisms are functions of type `:: a -> Writer a`. We'll define composition as `>=>`

```haskell
(>=>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
m1 >=> m2 = \x ->
    let (y, s1) = m1 x
        (z, s2) = m2 y
    in (z, s1 ++ s2)
```

We'll define an identity morphism, called `return`

```haskell
return :: a -> Writer a
return x = (x, "")
```

An example of how we could use this category for logging function calls.

```haskell
upCase :: String -> Writer String
upCase s = (map toupper s, "upCase ")

toWords :: String -> Writer [String]
toWords s = (words s, "toWords ")
```

Notice that the two functions modify their inputs differently, but still return a `Writer` category. This means that
despite the difference, we can still compose them. This acts as a fitting abstraction, as we can have even more complex functions
but still concatenate and log as long as we return a `Writer` type.

```haskell
process :: String -> Writer [String]
process s = upCase >=> toWords
```

So what we've done is not only allowed for composition, but added extra embellishments to our types thereby extending
functionality. This type of category is known as a Kleisli category. In our context, our objects are types for the our programming
language. Morphisms are functions that a map a type $A$ to an embellished type $B$.

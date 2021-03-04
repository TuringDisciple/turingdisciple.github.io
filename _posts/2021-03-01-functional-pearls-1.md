---
layout: post
title: The Smallest Free Number - Pearls of Functional Algorithm Design
categories: [Algorithms, Functional Programming]
---
**This blog post acts as expository notes on the content in the
text _Pearls of Functional Algorithm Design_ by Richard Bird for myself
and perhaps yourself, the reader. All source examples are in Haskell**

These imports will become useful.

``` haskell
import Data.Array
import Data.Array.ST
import Data.List (partition)
```

# The Smallest Free Number

The problem is as follows: compute the smallest natural number not in a given
finite set X of natural numbers.

The solution is dependent on the representation of X. If we have a set
that's sorted, then we can simply search for the first gap. But in an
unsorted list, we can't rely on a linear search.
It turns out that there exists a linear time solution to the
problem despite the fact that linear time solutions to list sorting
don't exist.

## Array Based Solution
We can specify the problem as the following.

```haskell
minFree :: [Int] -> Int
minFree xs = head ([0..]\\xs)
```

We use Int so as to only rely on core Haskell without having to import
a package. I could also implement a Nat datatype manually, but that would
be extra work distracting from the main purpose of this problem.
`xs\\ys` denotes the list of elements of xs that remain after removing
any elements in ys.

```haskell
(\\) :: Eq a => [a] -> [a] -> [a]
xs \\ ys = filter (`notElem` ys) xs
```

This solution runs with worst case $n^2$ complexity on a list of length
$n$. For example, minFree $[n-1, ..., 0]$ needs evaluating all  $i \notin [n-1, ..., 0]$
for $0 \le i \le n$ which is $n(n+1)/2$ comparisons.

The above solution relies on that not every number in the range
$[0 .. length\ xs]$ can be in xs.

The above solution conceptually builds a boolean array with n+1 slots
numbered from 0 to n, whose entries are initially False. For each element
$x \in xs$, with $x \le n$, we set the position $x$ in $xs$ to True.
minfree can then be considered to be the position of the first false
entry

``` haskell
minFree' = search.checklist
search :: Array Int Bool -> Int
search = length . takeWhile id . elems
```

Search in the above context takes a fixed size array of booleans, and
converts it into a list of booleans and the returns the length
of the longest initial segment consisting of True entries.

We can implement `checklist` in linear time using the function
`accumArray` in `Data.Array`.

```haskell
accumArray :: Ix i => (e -> a -> e) -> e -> (i, i) -> [(i, a)] -> Array i e
```

A rundown of this type signature is as follows:

- The type constraint `Ix i` restricts the type `i` to an index types. These
are types consisting of a contiguous subrange of values which
can be mapped to integers. This will be used to name indices
in our array.
- `(e->a->e)` is an accumulating function which will be used to transform
array entries and values into new entries,
- our second argument is an initial entry for each index,
- the third argument is a pair naming the upper and lower indices,
- the foruth is an association list of index-value pairs.

What this function will do is build an array by processing the association
list from left to right, combining entries and values into new entries using
the accumulating function. This is a linear time process in terms of the length
of the association list assuming a constant time association function.

N.B. An association list can be thought of a as a dictionary made from a linked-list
of key-value pairs. The value is *associated* with the key. Search is sequential
to find a value associated with a key.

We now define checklist

```haskell
checklist :: [Int] -> Array Int Bool
checklist xs = accumArray (||) False (0, n)
                      (zip (filter (<= n) xs ) (repeat True))
                      where n = length xs
```

`accumArray` can be used to sort a list of numbers in linear time *if*
elements are in a known range e.g. $(0, n)$. We to do this, let's define a
similar function to checklist called countlist

```haskell
countlist :: [Int] -> Array Int Int
countlist xs = accumArray (+) 0 (0, n) (zip xs (repeat 1)) where n = length xs
```

then we can define a sort function

```haskell
sort xs = concat [replicate k x | (x, k) <- assocs $ countlist xs]
```

Using `countlist` we can implement `minFree` as the position of the first $(x, 0)$ entry.

Another way of implementing checklist is to tick off entries using a constant
time update operation. This can be done using a monad encapsulating the operations.

```haskell
checklist' :: [Int] -> Array Int Bool
checklist' xs = runSTArray (do
                      a <- newArray (0, n) False
                      sequence [writeArray a x True | x <- xs, x <= n]
                      return a)
                      where n = length xs
 ```

What's going on here in our do? Well it looks quite similar to an imperative program.
The first line constructs an array with every value in our range $(0, n)$ initialised
to false. We then perform a list comprehension where we rewrite all values in a
to true if they are within xs. We require sequence as this transforms our list
of monads into monad with results. We then return a, our updated array. `runSTArray`
is a way to create a mutable array before returning an immutable result. In particular
we avoid copying the array for modification before returning it.

## Divide and Conquer solution

We now turn to a divide and conquer solution. The idea is summed up by
the expression minfree (xs ++ ys) $\equiv$ minfree xs ++ minfree ys. We first note some properties of `\\`. Remember the definition of `\\`:

```haskell
(\\) :: Eq a => [a] -> [a] -> [a]
xs \\ ys = filter (`notElem` ys) xs
```

with the following properties

```haskell
(as ++ bs) \\ cs = (as \\ cs) ++ (bs \\ cs)
as \\ (bs ++ cs) = (as \\ bs) \\ cs
(as \\ bs) \\ cs = (as \\ cs) \\ bs
```

These aren't proven for brevity. These laws are similar to laws of union in set theory where
`++` is union and `\\`is difference.
Suppose that `as` and `vs` are disjoint i.e `as \\ vs = as`, and
also `bs` and `us` are disjoint so `bs\\us = bs`. It follows that

```haskell
(as ++ bs) \\ (us ++ vs) = (as \\ us) ++ (bs \\ vs)
```

Lets choose natural number $b$ and let `as = [0..b-1]` and `bs = [b..]`.
Let `us = filter (< b) xs` and `vs = filter (>= b) xs`.

```haskell
[0..]\\xs = ([0..b-1]\\us) ++ ([b..] \\ vs) where (us, vs) = partition ( < b) xs
```

Partition is an efficient function for dividing a list.

Since `head (xs ++ ys) = if null xs then head ys else head xs` we can conclude for natural
number $b$

```haskell
minfree xs = if null ([0 .. b-1] \\ us)
             then head ([b .. ] \\ vs)
             else head ([0 .. ] \\ us)
             where (us, vs) = partition (< b) xs
```

We can speed up some of the above operations. For example the check
`null ([0..b-1]\\us)` can be performed in costant time assuming `us` is a list
of distinct natural numbers as well as every element is less than $b$:

```haskell
null ([0..b-1]) \\ us ≡ length us == b
```

We can generalise `minfree` to a function, which we call `minfrom`

```haskell
minfrom :: Int -> [Int] -> Int
minfrom a xs = head ([a..]\\xs)
```
where every element of `xs` is assumed to be greater than or equal to `a`. Then provided we select
$b$ so that both `length us` and `length vs` are less than `length xs`:

```haskell
minfree xs = minfrom 0 xs
minfrom a xs | null xs            = a
             | length us == b - a = minfrom b vs
             | otherwise          = minfrom a us
             where (us, vs) = partition (< b) xs
```

So how do we select $b$? Ideally our value of $b$ is such that the maximum of `length us` and `length vs` is as small as possible *and* $b\gt a$.

$$
b = a + 1 + n\ div\ 2
$$

where div is floored division.

If $n \ne 0$ and `length us < b − a`, then
`length us ≤ n div 2 < n`
And, if `length us = b − a`, then `length vs = n − n div 2 − 1 ≤ n div 2`

With this choice the number of steps $T(n)$ for evaluating `minfrom 0 xs
when n = length xs` satisfies $T(n) = T(n\ div\ 2) + Θ(n)$, with the solution
$T(n) = Θ(n)$.

A final optimisation is we can avoid computing length by represent xs as a pair `(length xs, xs)`. Our final solution then looks like the following:

```haskell
minfree xs = minfrom 0 (length xs, xs)
minfrom a (n, xs) | n == 0 = a
                  | m == (b-a) = minfrom b (n-m, vs)
                  | otherwise = minfrom a (m, us)
                  where (us, vs) = partition (<b) xs
                        b        = a + 1 + (n `div` 2)
                        m        = length us
```

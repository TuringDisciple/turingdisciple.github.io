---
layout: post
title: A Surpassing Problem - Pearls of Functional Algorithm Design
categories: [Algorithms, Functional Programming]
---
**This blog post acts as expository notes on the content in the
text _Pearls of Functional Algorithm Design_ by Richard Bird for myself
and perhaps yourself, the reader**

This problem was proposed by Martin Rem in 1988.

We define the surpasser of an element in an array to be the
element `x[j]` such that for element `x[i]`, `x[i] < x[j]`
for $i \lt j$. From this definition, we define the surpasser
count to be the number of surpassers in an array. Here's an illustrative
example using the ASCII string GENERATING

```
    G E N E R A T I N G
    5 6 2 5 1 4 0 1 0 0
```

this is assuming ASCII encoding of letters, i.e. A=41, B=42 ...
The problem is to compute the maximum surpasser count
of an array length $n > 1$, and to do so in $\mathcal{O}(n\log n)$
time.

## Naive Solution
We represent our input as a list, and specify the problem:

```haskell
msc :: Ord a => [a] -> Int
msc xs = maximum [scount z zs | z:zs <- tails xs]
scount x xs = length (filter (x < ) xs)
```

`scount` caclulates the surpasser count for an element x in xs.
Tails will return the non-empty tails of a list

```haskell
tails :: [a] -> [[a]]
tails [] = []
tails (x:xs) = (x:xs):tails xs
```

This solution is quadratic. `scount` will perform
with worst case $\mathcal{O}(n^2)$ complexity. `maximum`
performs with $\theta(n)$ complexity.


## Divide and Conquer Solution
Our target complexity is $\mathcal{O}(n\log n)$. If we
can find a divide and conquer solution such that

```haskell
msc(xs ++ ys) = join (msc xs) (msc ys)
```

with `join` computing in linear time, then our solution will
perform with complexity $T(n) = 2T(n/2) + \mathcal{O}(n)$, our solution
would be $T(n) = \mathcal{O}(n\log n)$. We don't have enough
information yet however to define this linear join. Suppose we build
a table of all the surpasser counts

```haskell
table xs = [(z, scount z zs) | z:zs <- tails xs]
```

Then `msc = maximum.map snd.table`. With this table, we can try and
find a linear join s.t.

```haskell
table (xs ++ ys) = join (table xs) (table ys)
```

There's a property of tails that can assist in solving this problem:

```haskell
tails (xs ++ ys) = map (++ys) (tails xs) ++ tails ys
```

This leads to the following derivation

```haskell
 table (xs ++ ys)
 = -- {definition-}
 [(z , scount z zs) | z : zs <- tails (xs ++ ys)]
 = -- {divide and conquer property of tails}
 [(z , scount z zs) | z : zs <- map (++ys) (tails xs) ++ tails ys]
 = -- {distributing ← over ++ & desugaring map}
 [(z , scount z (zs ++ ys)) | z : zs <- tails xs] ++
 [(z , scount z zs) | z : zs <- tails ys])
 = -- {since scount z (zs ++ ys) = scount z zs + scount z ys}
 [(z , scount z zs + scount z ys) | z : zs <- tails xs] ++
 [(z , scount z zs) | z : zs <- tails ys])
 = -- {definition of table and ys = map fst (table ys)}
 [(z , c + scount z (map fst (table ys))) | (z , c) <- table xs] ++ table ys
```

This means we can define join as the following

```haskell
join txs tys  = [(z, c + tcount z tys) | (z, c) <- txs] ++ tys
tcount z tys = scount z (map fst tys)
```

This isn't a linear time solution however as join will not perform in
linear time due to the `tcount` operation. If the input to tcount is sorted in ascending
order we can reason the following

```haskell
tcount z tys
=  -- {definition of tcount and scount}
length (filter (z <) (map fst tys))
=  -- {since filter p · map f = map f · filter (p · f )}
length (map fst (filter ((z <) . fst) tys))
=  -- {since length · map f = length}
length (filter ((z <) . fst) tys)
=  -- {since tys is sorted on first argument}
length (dropWhile ((z >=) . fst) tys)
```

Therefore

```haskell
tcount z tys = length (dropWhile ((z >=).fst) tys)
```

With this calculation, we need table to store values in
ascending order of the first component.

```haskell
table xs = sort [(z, scount z zs) | z:zs <- tails xs]
```

Performing the above calulation of table we get the following
property.

```haskell
join txs tys = [(x , c + tcount x tys) | (x , c) <- txs] ^^ tys
```

where `^^` is an operation to merge two sorted lists.

We derive a recursive efficient version of `join`.

```haskell
join txs [] = txs
join [] tys = tys
join txs@((x, c):txs') tys@((y, d):tys') =
    ((x, c + tcount x tys):[(x, c + tcount x tys) | (x, c) <- txs']) ^^ tys
```

To perform `(^^)` we need to merge compare $x$ and $y$. In the
case that $x \lt y$, then it's the element on the left and
with `tcount x tys = length tys`, this reduces to.

`(x, c + length tys):join txs' tys'`

If $x = y$, we compare `c + tcount x tys` and `d=tcount x tys'`. In this case this
reduces

`(y, d): join txs tys'`.

This result is also true for $x > y$.

Our final algorithm becomes the following, we add amother argument to join to avoid
recalculating length:

```haskell
msc :: Ord a => [a] -> Int
msc = maximum.map snd.table
table [x] = [(x, 0)]
table xs = join (m - n) (table ys) (table zs)
              where m = length xs
                    n = m `div` 2
                    (ys, zs) = splitAt n xs
join 0 txs [] = txs
join n [] tys = tys
join n txs@((x, c):txs') tys@((y, d):tys')
      | x < y  = (x, c + n):join n txs' tys
      | x >= y = (y, d):join (n - 1) txs tys'
```

With linear join, we have a $\mathcal{O}(n\log n)$ algorithm.

---
layout: post
title: Saddleback Search - Pearls of Functional Algorithm Design
category: Functional Programming
---
**This blog post acts as expository notes on the content in the
text _Pearls of Functional Algorithm Design_ by Richard Bird for myself
and perhaps yourself, the reader. All source examples are in Haskell.**

The problem is stated as the following. Suppose
we are given a function $f:\mathbb{R}\times\mathbb{R} \to\mathbb{R}$.
The value of `invert f z` is a list of all pairs $x, y$ s.t.
$f(x, y) = z$. We can assume that f is strictly increasing in
each argument, but nothing else.

We can approach this problem naively; knowing that the arguments
to our function are strictly increasing, we can imply that
$x,y\le z$. Hence we can perform a list comprehension search.

```haskell
invert :: (Int -> Int -> Int) -> Int -> [(Int, Int)]
invert f z = [(x, y) | x <- [0..z], y <- [0..z], f x y == z]
```

This isn't very efficient as we perform $(z+1)^2$ evaluations
of our function. We can actually place a stronger bound
on our search space of values, by knowing that $f(x, y) \ge x + y$
following from our stricly increasing assumption. This leads to the
following:

```haskell
invert f z = [(x, y) | x <- [0..z], y <- [0..z - x], f x y == z]
```

This bounds us to $z(z-x)$ searches, which is still quadratic, but a
slight improvement.

Our problem is starting to take the shape of searching
values within a grid, that satisfy a property defined
by our function:

```
    (0, z)
        ______________ (z, z)
        |              |
        |              |
        |              |
        |              |
        |              |
        |______________|
    (0,0)              (z, 0)

```

Visually, this illuminates our search space.
Our initial solution was equivalent to iterating
over $x$, and searching each column $(x, 0\to z)$
for our solution set. Our second solution bounded
our search space and reduced our problem to
searching $(0\to x, 0\to z-x)$.

We can search our space in a different order, and
potentially gain something. Suppose we start
our search from $(0, z)$, what do we get?

```
(0, z)
     ______________ (z, z)
    |              |
    |    (u,v)     |
    |      ________|
    |      |       |
    |      |       |
    |      |       |
    |______|_______|
(0,0)              (z, 0)

```

What is useful is that with each step we constrain
our search space.

```haskell
find :: (Int, Int) -> (Int -> Int -> Int) -> Int -> [(Int, Int)]
find (u, v) f z  = [(x, y) | x <- [u..z], y <- [v, v-1, .. 0], f x y == z]
invert f z = find (0, z) f z
```

This constrains the number of checks we perform to $z^2$, which isn't
an improvement, but invites us to a different mode of thinking of our
problem and improving `find`.

1. If we evaluate our function on our starting
point ,  $f(u, v)$, and find it to be less than $z$, we can actually skip
searching $v' \lt v$ as we know that $f(u,v) \lt z \implies f(u, v'\lt v) \lt z$.

2. In the case where $f(u, v) > z$, we can try find a closer value by decreasing
our search in the column i.e. we search $f(u, v' \lt v)$.

3. In the case where $f(u, v)\equiv z$, we can skip searching our columns and row.

The above facts lead us to the following improved definition of `find`

```haskell
find (u, v) f z
      | u > z || v < 0 = [] -- values outside our square
      | z' < z         = find (u + 1, v) f z        -- condition 1
      | z' > z         = find (u, v - 1) f z        -- condition 2
      | z' == z        = (u, v):find (u+1, v-1) f z -- condition 3
      where z' = f(u, v)
```

With the above definition we have a best case of only $z+1$ evaluations,
this corresponds to the case where we simply proceed to the bottom or
rightmost boundary. In the worst case we have $2z + 1$ evaluations
where find traverses the perimeter of the square from $(0, z)$ to $(z, z)$
then proceeds downward to $(z, 0)$. This is a massive improvement in
performance as we now have a linear time solution, all from switching
the order of our search.

Perhaps we can constrain our search space even further, knowing that $x+y\le z$.
If we know the maximum values of $x$ and
$y$ that satisfy the equation $f(x_{max}, m), f(n, y_{max})=z$, then we
can constrain search to values $n\le x_{max}\le z$ and $m\le y_{max}\le z$.
Finding these maximum values would constrain our search space to $(x_{max}+1)(y_{max}+1)$
and is essentially finding a starting rectangle. These maximum values can
be found via binary search.

```haskell
bsearch g (a, b) z
      | a + 1 == b  = a
      | g m <= z    = bsearch g (m, b) z
      | otherwise   = bsearch g (a, m) z
      where m = (a + b) `div` 2
ymax z f = bsearch (\y -> f(0, y)) (-1, z+1) z
xmax z f = bsearch (\x -> f(x, 0)) (-1, z+1) z
```

In the above snippet, $g$ is an increasing function on the natural
numbers. We have extended the range that $f$ operates over so that
$f(0, -1), f(-1, 0) = 0$. With these values of $x_{max}$ and
$y_{max}$, our invert function will take $2\log z + x_{max} + y_{max}$
evaluations in the worst case and $2\log z + min(x_{max}, y_{max})$ in
the best case. If $x, y$ are substantially smaller then $z$, we essentially have $\mathcal{O}(\log z)$
complexity algorithm.

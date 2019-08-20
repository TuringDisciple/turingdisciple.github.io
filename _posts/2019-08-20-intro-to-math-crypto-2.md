---
layout: post
title: " Discrete Logarithms and Diffe-Hellman"
---

# Introduction
These are just some notes on the discrete logarithm problem and Diffe-Hellman taken from chapter 2 of Introduction to Mathematical Cryptography

# Formal definition
Let $g$ be a primitive root for $F_{p}$, and h be a non-zero element of this field, then the DLP is finding an exponent x st

$$ 
      g^{x} = h (mod\ p)
$$

x is called the discrete logarithm of h to the base g and is denoted $log_{g}(h)$

This definition can extend to a group. Given a group $G$ with group operation *, the DLP for the group is simply for any two given elements $g$ and $h$, find $x$ such that

$$
      g ^{x} = h
$$

# Diffe-Hellman Key Exchange
The Diffie-Hellman key exchange algorithm solves the problem of exchanging secure information over an insecure channel. In particular we want to share a secret key for encryption over an insecure channel for a symmetric cipher.

## Formal defintion
Two parties Akice and Bob agree on a large prime $p$ and non-zero integer $g$ modulo $p$. Alice and bob make these values public knowledge. Then Alice and Bob respectively compute the following. Alice selects private integer $a$ and Bob selects private integer $b$

$$
A = g ^ {a} (mod\ p)\\
B = g ^ {b} (mod\ p )
$$

They then exchange $A$ and $B over the insecure channel and compute the following respectively

$$
A' = B ^{a}(mod\ p)\\
B' = A^ {b}(mod\ p)
$$

Note that $A' = B'$ proof below

$$
A' = B ^ {a} = g^{ba} = g ^ {ab} = A^{b} = B' 
$$

This value $A'(B')$ is the the private key for the two schemes
The security of the DHP comes from the difficulty of the Diffe-Hellman problem. This problem is defined as the difficulty of computing $g^{ab} (mod p)$ from $g^{a} (mod p)$
and $g^{b} (mod p)$

If the DLP can be solved, then it's trivial to solve the DHP. However it is not known whether the converse is true. If an attacker can solve the DHP, can they use that algorithm for the DLP.

# ElGamal
The Diffe-Hellman key exchange unfortunately doesn't act as a full PKE scheme as it doesn't allow for the transfer of specific messages but rather random bits. The ElGamal encryption scheme uses the difficulty of the DLP to enable encryption.

The formal definition of the scheme is as follows. Alice selects a screte number $a$ to act as her private key and she computes

$$ 
 A = g ^ {a} (mod\ p)
$$

This is the public key. This is used by Bob to encrypt a message which is between $m \leftarrow 2 ... p$. To encrypt the message Bob selects a number $k$ modulo $p$. Bob uses $k$ to encrypt only one message, then discards the key. The number k is called an ephemeral key. Bob takes all these values and computes the following

$$
c_1 = g^k (mod\ p)\\
c_2 = mA^k(mod\ p)
$$

The encryption that Bob generates is this pair of numbers $(c_1, c_2)$. For Alice to decrypt she first computes $x \leftarrow c_{1}^{a} (mod \ p)$. And then computes the message $m \leftarrow x^{-1}\cdot c_2$. A proof as to why this encryption works is as follows

$$
x = c_1^a(mod\ p) = g ^ {ak}(mod \ p) = A^k\\
x^{-1} \cdot c_2 = A^{-k}mA^k (mod\ p) = m (mod\ p)
$$

# Hardness of the DLP 
NOTE: a primer on order notation. We say that a function $f(x)$ is big-$O$ $g(x)$ if there exists positive constants $c$ and $C$ such that

$$
 f (x) \leq cg(x) \  \forall x \geq C
$$

Alternatively one could understand big-$O$ notation in terms of taking limits. In paritcular if the limit of the following exists

$$
\lim_{x \to \infty} \frac{f(x)}{g(x)}
$$

then $f(x)=O(g(x))$.

We start with our original dlp $g^x = h$ in $G=\mathbb{F}^*_p$. If we select $p$ to be between $2^k$ and $2^{k+1}$, $g, h$ and $p$ all require at most $k$ bits. Then we can solve the original dlp in $O(p)=O(2^k)$ time using a trial and error method. This is exponential time.

Considering the DLP in the group $G=\mathbb{F}_p$, where the group operation is addition, the DLP in this case asks for a solution $x$ to the congruence $x\cdot g\equiv h \ (mod \ p)$

By using the extended Euclidean algorithm, to compute $g^{-1} (mod \ p)$ , you can compute $x = g^{-1} \cdot h (mod \ p)$.
This takes $O(log\ p)$ steps. 

NOTE: notation
$\mathbb{Z}/p\mathbb{Z}$ and $\mathbb{F}_p$ denote the finite field defined as below
$$
\mathbb{F}_p = \{0, 1, 2, ..., p-1\}
$$
This is the ring of integers modulo $p$. 

$(\mathbb{Z}/p\mathbb{Z})^*$

 and $\mathbb{F}^*_p$ denotes the group of units modulo $p$. 

$$
\mathbb{F}^*_p=\{a \in \mathbb{F}_p : gcd(a, p) = 1 \}
$$

It can be trivially seen from above that if $p$ is prime then the group $\mathbb{F}^*_p = \{1, .., p-1\}$.

# A Collision Algorithm for the DLP

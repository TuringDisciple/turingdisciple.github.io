---
layout: post
title: "Information theory basics"
--- 

# Introduction
Recently I have had the opportunity to learn a bit about information theory. Here are my crash course notes on the basic idea behind information theory. To learn more, Google is a great resource and so is the [original paper](http://www.math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf) written by Shannon in the late 40s which proposed the theory. This is one of the most exciting mathematical concepts especially when it comes to wide reaching applications in computer science.

With information theory we are trying to understand the amount of information that can be communicated across a channel by only measuing the statistics of data travelling along that channel.

# Shannon's entropy
The theory of information starts with an attemp to allow us to quantify the informativeness of information, but not it's salience or validity.

Shannon's entropy is a single quantity that measures this idea of informativeness, balancing how useful a pieve of information is with how likely you are to get it.

Let $X$ be random variable for a finite discrete distribution of size $n$, and a probability mass function $p_{X}$ giving probabilities $p_{X}(x_i)$, the entropy is

$$ 
H(X) = - \sum_{x_i \in X} p_X(x_i)\log_2p_X(x_i)
$$

The first thing to note is that this definition works for any sample space. $H(X)$ is always defined for all sample spaces, as all sample spaces have a defined probability for each element. A prpoperty of Shannon's entropy is that 

$$
H(X) \geq 0
$$

This follows from $0\geq p_X(x_i) \leq 1$. Furthermore it can be trivially proven that if all events are equally likely then 

$$ 
H(X) = \log \#(\mathcal X)
$$

It can be proven also that

$$
H(X) \leq \log \# \mathcal(X)
$$

This essentially means that the most informative an experiment can be is when we have no idea before the experiment of what the outcome is, which is essentially when every possibility is equally likely.

Furthermore if $p_X(x_k) = 1$ for some $k$, then $H(X) = 0$. This makes sense in that we have absolute certainty in an outcome meaning no information is gained by viewing the outcome.

The source coding theorem shows that the entropy $H(X)$ is a lower bound on the average length of a message using the most efficient code; it is a limit on the compressibility of the data. 

# Joint entropy and conditional entropy
The joint entropy is just the entropy pf the joint distribution:

$$
H(X, Y) = - \sum_{x, y}p_{X,Y}\log_{2} p_{X, Y}(x, y)
$$


we can intuitively think of the conditional probabilty as the probability of a value multiplied by the conditional of another value when given the initial one i.e $p_(x, y) = p(x|y)p(y)$.

The conditional entropy can then be given as follows

$$
H(X|Y=y) = - \sum_x p_{X|Y}(x|y)\log_2 p_{X|Y}(x|y)
$$

This is the entropy of the variable $X$ if we know $Y=y$. The conditional probability is the **average** of this, averaged of all the values of $y$

$$
H(X|Y) = - \sum_y p_Y(y)H(X|Y=y) = - \sum_{x, y}p_(X, Y)\log_2 p_{X|Y}(x|y)
$$ 

If $X$ and $Y$ are independent, then $H(X|Y) = H(X)$. Conversely if $X$ is determined by $Y$ i.e. $x = f(y)$, then in this case

$$
H(X|Y) = 0
$$

Intuitvely we can think of $H(X|Y)$ to be the average amount of information within $X$ given that we know $Y$.

There's also the chain rule of entropy

$$
H(X, Y) = H(X) + H(Y|X)
$$

this intuitively makes sense as it describes how the joint information will be the information in $X$ as well as the information left-over in $Y$ when we have $X$.

# Mutual information
The mutual information is the measure of how related two distributions are, it is given by

$$
I(X, Y) = H(X) + H(Y) - H(X, Y)
$$

Thus it is the amount of information in X and Y considered separately, minus the amount of information in them considered together.

If the two distributions are independent $I(X, Y) = 0$, conversely if $Y$ is determined by $X$, then $H(Y|X) = 0$ and $H(X,Y) = H(X)$ and $I(X, Y) = H(Y)$.
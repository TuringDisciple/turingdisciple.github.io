---
layout: post
title: "Paper Write-up: Generative Adversarial Nets (2014)"
---

_I want to start doing blog posts on papers that I am currently reading, or papers that I love. This is the first in the series and it is the paper which proposed the GAN, one of the most exciting neural network models around that has gotten quite a lot of attention due to results._

# What is this about? 
Notes on the the generative adversarial nets paper by [Goodfellow et.al](https://arxiv.org/abs/1406.2661). 

## Introduction
The basic idea of the adversarial nets framework is that we have a generative model which is pitted against an adversary: a discriminative model that learns to determine whether a sample is from the model distribution or the data distribution. 

A good analogy for this network is imagine a counterfeiter pitted against anti-fraud police. Ultimately the criminal party is trying to create currency that is indistinguishable from the real thing. 

A benefit of these networks is that there's no dependence on approximate inference and Markov chains. This model is simply trained using back-propagation and sampled using forward propagation.

## Formal Definition
To learn the generators distribution $p_g$ over data $\mathbf{x}$ we a define a prior on input noise variables $p_{\mathbf{z}}(\mathbf{z})$ thene represent a mapping to data space as $G(\mathbf{z}; \theta_g)$ where $G$ is a differentiable function represented by a multilayer perceptron (MLP) with parameters $\theta_g$. We define a second MLP $D(\mathbf{x};\theta_d)$ that outputs a single scalar $D(\mathbf{x})$ that represents the probability that $\mathbf{x}$ came from the data rather than from $p_g$. $D$ is trained to maximize the probability of assigning the correct label to both training examples and samples from $G$. We simultaneously train $G$ to minimize $\log(1-D(G(\mathbf{x})))$.

$D$ and $G$ play the following minimax game with value function $V(G, D)$

$$
\min_G \max_G V(G, D) = \mathbb{E}_{\mathbf{x}\sim p_{data}(\mathbf{x})}[\log\ D(\mathbf{x})] + \mathbb{E}_{\mathbf{z}\sim p_{\mathbf{z}}(\mathbf{z})}[\log\ (1-D(G(\mathbf{z})))]
$$

The problem with the above function is that it may not provide a sufficient gradient for $G$ to learn well. Early in the learning process when $G$ still provides poor samples, $D$ can reject samples with high confidence because they're clearly different from the training data. Rather than train $G$ to minimize $\log\ 1 - D(G(\mathbf{x}))$ we can train it to maximize $\log\ (D(G(\mathbf{z})))$.

![Prob Distribution](/assets/gans-prob.png)

The figure above illustrates pictorially the idea behind the training process (taken from the source paper). We have five lines: the black dotted line corresponds to the true distribution $p_{data}$. The green line corresponds to the probability of the generative model $p_{g}$. The blue dotted line corresponds to the discriminator probability. The lower horizontal line corresponds to the space in which $\mathbf{z}$ is sampled and the upper horizontal line is the sample space of $\mathbf{x}$. The upwards arrows show the function $\mathbf{x}=G(\mathbf{z})$.

As can be seen from the figure, prior to any training let's consider the case in which $p_{G}$ is near convergence (a). As can be seen in $a$, the discriminator has no certain accuracy, and the generator isn't producing convincing samples. (b) shows that $D$ is trained to discrimanate samples from dat, converging to $D^*(\mathbf{x}) = \frac{p_{data}(\mathbf{x})}{p_{data}(\mathbf{x}) + p_g(\mathbf{x})}$. (c) shows that after $G$ is updated, the gradient of $D$ has caused $G$ to flow to regions that are more likely. (d) shows the converged case, where after enough training steps, a point is reached which cannot be improved as $p_g = p_{data}$. The discriminator is unable to differentiate between the two probabilities and therefore $D(x) = \frac{1}{2}$.

## Theoretical Results
In this section of the paper, Goodfellow et.al define the algorithm which is used to train the model, as well as showing that the minimax game has a global optimum for $p_g=p_{data}$ and that the algorithm provided optimizes the min-max game function above. I would advise, if interested, to read through the proofs of the above in the [original paper](https://arxiv.org/abs/1406.2661). Below is a screen-shot of the algorithm definition taken from the paper: 

![Algorithm 1](/assets/gans-alg.png)

## Experiments
Within this part of the paper, the authors show results of their network when trained on different generative model tasks such digit generation, and face and image generation. Again I would advise the reader to look at their results, although it's worth noting that since the publishing of the paper GANs have been used to obtain much better [(and sometimes frightening)](https://www.youtube.com/watch?time_continue=1&v=kSLJriaOumA) results.

## Advantages and Disadvantages
### Disadvantages
- No explicit representation of the generative probability
- $D$ must be synchronized with $G$ during training so that $G$ is not trained too much without updates to $D$
- 
### Advantages
- No need for Markov chains (only backprop used for training)
- No need for inference during learning
- Supports a wide range of functions
- Can represent "sharp" distributions
- Generator network not updated with data examples directly but only with gradients flowing through the discriminator.


# Conclusion
Although this paper was released in 2014, it's relevance can still be felt today through it's many extensions. I hope I was able to shine a light on the GANs and the world therein.
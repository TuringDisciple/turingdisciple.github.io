---
layout: post
title: "Solving Textual Entailment with The DecAtt model"
---
# Introduction
In this post I will write about the problem of textual entailment and how one of my favourite models has been used to tackle the problem.

# What is Textual Entailment?
The problem of textual entailment is simple. Given a premise $p$ and a hypothesis $q$, 
can you infer the logical relationship between the pairs. $p$ and $q$ are assumed 
to be grammatically correct sentences and the inferred logical relationship is in one 
of 3 different classes. 

1. Entailment: $p\rightarrow q$
2. Contradiction $\neg p\rightarrow q$
3. Neutral: cannot infer any logical relationship

An example shows the problem a bit more clearly. Given the sentence $\textit{"President Trump visited the Detroit, Michigan"}$ an example of sentences which would meet the three categories is given below

1. Entailing sentence: $\textit{The President was in a US state}$
2. Contradicting sentence: $\textit{The President has never left the white house}$
3. Neutral sentence: $\textit{Preying mantises have been shown to be cannibalistic}$

# Research Attempts
It turns out that this problem within the field of NLP can be tackled with deep learning and machine learning methods. Some of the more interesting attempts that have come to solve this problem over the past several years will be discussed alongside the DecAtt model.

## Datasets
There are currently 3 primary datasets that have been used to benchmark performance on the textual entailment task. 

1. [SICK](https://pdfs.semanticscholar.org/1fd8/0b5adeec4d5e921c7499a50c2cfc5b9686ad.pdf): this was proposed as the dataset to use for the 2014 SemEval task 1 dataset. No longer used as the standard dataset, it did serve a purpose as putting forward the textual entailment task in a more formalised matter. This features 10,000 sentence pairs/
2. [SNLI](https://nlp.stanford.edu/projects/snli/): most likely considered the golden standard for the textual entailment task. This dataset features 500,000 sentence pairs, annotated and accuracy of model prediction is the primary metric. 
3. [MNLI](https://www.nyu.edu/projects/bowman/multinli/): another standard dataset, can be used in conjunction with SNLI dataset. The reason why this dataset differs from SNLI is that it deals with multi-genre which adds a layer of difficulty compared to SNLI as all SNLI assumes all text assumes the same style.

## DecAtt (2016)
Proposed in [2016 by Parikh et.al](https://arxiv.org/pdf/1606.01933v1.pdf), this model is notably worth discussing due to it's high performance against its very low complexity. Compared to the [PWIM](https://www.aclweb.org/anthology/N16-1108) model which uses almost a whole factor of 10 more parameters, it outperforms this model considerably. 

The model works by using several feed-forward neural networks, which serve to attend on different aligned sub-parts of phrases. First by converting input sentences to 200D input encoding, the pairs of sentences go through three steps. 

 *Attend*: the attend step is the first step and what it does is it takes the input sentences represented as $\mathbf{a}=(a_1, ..., a_{l_a})$ and $\mathbf{b}=(b_1, ..., b_{l_a})$ and using a variant of neural attention and decomposes the problem into a comparison of aligned sub-phrases. First an un-normalized attention matrix with entries denoted as $e_{ij}$ is computed as the following 
 $$
      e_{ij} = F(a_i)^TF(b_i)
 $$
 It should be noted that in the above equation, $F$ is a feed-forward neural network with ReLU activations. The attention weights are then normalised as follows

 $$
      \beta_i = \sum^{l_b}_{j=1} \frac{exp(e_{ij})}{\sum^{l_b}_{k=1}exp(e_{ik})}b_j\\
     \alpha_j = \sum^{l_a}_{i=1} \frac{exp(e_{ij})}{\sum^{l_a}_{k=1}exp(e_{kj})}a_i\\
       
 $$

 $\beta_i$ is the subphrase in $\mathbf{b}$ softly aligned to $a_i$ and vice-versa for $\alpha_i$.

*Compare*: In the compare step, aligned phrases are separately compared. 

$$
\mathbf{v}_{1, i} := G([a_i, \beta_i])\ \ \forall i \in [1, ..., l_a]\\
\mathbf{v}_{2, j} := G([b_j, \alpha_j]) \ \ \forall i \in [1, ..., l_b]
$$

Note that in the above $G$ is another feed-forward neural network. $[., .]$ brackets denote concatenation.

*Aggregate*:
There are now two comparison vectors $\mathbf{v_1}, \mathbf{v_2}$ . These are aggregated over

$$
\mathbf{v_1} = \sum^{l_a}_{i=1}\mathbf{v_{1, i}}\ \ \ \ \ , \ \ \ \ \ 
\mathbf{v_2} = \sum^{l_b}_{j=1}\mathbf{v_{1, j}}
$$

and fed through a final feed-forward neural network $H$ which provides a label array $\mathbf{y}\in \mathbb{R}^D$
$$
      \mathbf{y} = H([\mathbf{v}_1, \mathbf{v}_2])
$$

All other information regarding choice of performace, training parameters and computational complexity can be found in the original paper [here](https://arxiv.org/pdf/1606.01933v1.pdf). The paper reports **86%** accuracy on the SNLI dataset.

## Conclusion
What is interesting about this model is that infers logical relationships between sentence pairs by training a model to approximate attention weightings within a sentence pair matrix. This is not necessarily a unique idea, and models such as [ESIM](https://arxiv.org/pdf/1609.06038.pdf) try to do a similar thing. But these require a much greater computational load due their use of LSTMs. Another model, briefly mentioned, called [PWIM](https://www.aclweb.org/anthology/N16-1108) takes a novel approach to this problem by converting the problem into a computer vision problem by using a convolutional neural net.

In 2019, the best performing models for this task are [very large deep learning](https://paperswithcode.com/sota/natural-language-inference-on-snli) models of the BERT variety, but it is worth considering that there is still potential research to be done into simpler approaches. 



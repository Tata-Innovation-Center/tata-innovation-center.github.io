---
layout: default
title: Variational Autoencoder
mathjax: true
parent: Advanced topics in machine learning
nav_order: 4
date: 2020-02-20 12:00:00
---

# Variational Autoencoder
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Latent variable models

There are a lot of latent variables for a image x of a person, for example, the gender, eye color, hair color, pose, etc. And the idea of latent variable models(LVM) is to explicitly model these factors using latent variables z. 

A latent variable model defines a probability distribution

$$
p(x, z)=p(x | z) p(z)
$$

Challenge: Very difficult to specify the conditional distribution or the distribution of latent variables by hand.

Key idea:

1. Assume latent variable is from a gaussian distribution, $$\mathbf{z} \sim \mathcal{N}(0, l)$$
2. And the conditional distribution is also in some form of gaussian, $$p(\mathbf{x} | \mathbf{z})=\mathcal{N}\left(\mu_{\theta}(\mathbf{z}), \Sigma_{\theta}(\mathbf{z})\right)
$$ where $$\mu_{\theta},\Sigma_{\theta}$$ are neural networks

## Shallow mixture models

Mixture of Gaussians. 
1. $$
\mathbf{z} \sim \text { Categorical }(1, \cdots, K)
$$
2. $$
p(\mathbf{x} | \mathbf{z}=k)=\mathcal{N}\left(\mu_{k}, \Sigma_{k}\right)
$$

Generative process: 
1. Pick a mixture component k by sampling z
2. Generate a data point by sampling from that Gaussian

The likelihood is non-convex: this increases representational power, but
makes inference more challenging

$$
p(\mathbf{x})=\sum_{\mathbf{z}} p(\mathbf{x}, \mathbf{z})=\sum_{\mathbf{z}} p(\mathbf{z}) p(\mathbf{x} | \mathbf{z})=\sum_{k=1}^{k} p(\mathbf{z}=k) \underbrace{\mathcal{N}\left(\mathbf{x} ; \mu_{k}, \Sigma_{k}\right)}_{\text {component }}
$$

## Deep latent-variable models

### Key idea: A mixture of an infinite number of Gaussians 
1. $$\mathbf{z} \sim \mathcal{N}(0, l)$$ (From discrete to continuous)
2. $$p(\mathbf{x} | \mathbf{z})=\mathcal{N}\left(\mu_{\theta}(\mathbf{z}), \Sigma_{\theta}(\mathbf{z})\right)
$$ where $$\mu_{\theta}, \Sigma_{\theta}$$ are neural networks. 

$$
\mu_{\theta}(\mathbf{z})=\sigma(A \mathbf{z}+c)=\left(\sigma\left(a_{1} \mathbf{z}+c_{1}\right), \sigma\left(a_{2} \mathbf{z}+c_{2}\right)\right)=\left(\mu_{1}(\mathbf{z}), \mu_{2}(\mathbf{z})\right)
$$

$$
\left.\Sigma_{\theta}(\mathbf{z})=\operatorname{diag}(\exp (\sigma(B \mathbf{z}+d)))\right)=\left(\begin{array}{cc}{\exp \left(\sigma\left(b_{1} z+d_{1}\right)\right)} & {0} \\ {0} & {\exp \left(\sigma_{\left.2 z+d_{2}\right)}\right)}\end{array}\right)
$$

$$
\theta=(A, B, c, d)
$$

3. Even though $$p(\mathbf{x} \mid \mathbf{z})$$ is simple the marginal distributio of x is very complex/flexible


### Maximum likelihood learning for the above model:

$$
\log \prod_{\mathbf{x} \in \mathcal{D}} p(\mathbf{x} ; \theta)=\sum_{\mathbf{x} \in \mathcal{D}} \log p(\mathbf{x} ; \theta)=\sum_{\mathbf{x} \in \mathcal{D}} \log \sum_{\mathbf{z}} p(\mathbf{x}, \mathbf{z} ; \theta)
$$

Evaluating $$log \sum_{\mathbf{z}} p(\mathbf{x}, \mathbf{z} ; \theta)$$ can be hard because the space for latent variable z is huge.

Solution: Approximations.

### **First attempt**: Naive Monte Carlo
1. Sample $$z^{(1)}, \cdots, z^{(k)}$$ uniformly at random
2. Approximate expectation with sample average

$$
\sum_{\mathbf{z}} p_{\theta}(\mathbf{x}, \mathbf{z}) \approx|\mathcal{Z}| \frac{1}{k} \sum_{j=1}^{k} p_{\theta}\left(\mathbf{x}, \mathbf{z}^{(j)}\right)
$$

Works in theory but not in practice.  uniform random sampling is not a good choice.

### **Second attempt**: Importance Sampling
$$
p_{\theta}(\mathbf{x})=\sum_{\text {All possible values of } \mathbf{z}} p_{\theta}(\mathbf{x}, \mathbf{z})=\sum_{\mathbf{z} \in \mathcal{Z}} \frac{q(\mathbf{z})}{q(\mathbf{z})} p_{\theta}(\mathbf{x}, \mathbf{z})=\mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}\left[\frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})}\right]
$$
1. 1 Sample $$\mathbf{z}^{(1)}, \cdots, \mathbf{z}^{(k)}$$ from q(z)
2. Approximate expectation with sample average
$$
p_{\theta}(\mathbf{x}) \approx \frac{1}{k} \sum_{j=1}^{k} \frac{p_{\theta}\left(\mathbf{x}, \mathbf{z}^{(j)}\right)}{q\left(\mathbf{z}^{(j)}\right)}
$$

### Choice of q(z)
A new question now: what is a good choice of q(z)? how to derive algorithms for choosing q and extending this approximation to the marginal log-likelihood.

We can approximate marginal probabilities with importance sampling: 

$$
p_{\theta}(\mathbf{x}) \approx \frac{1}{k} \sum_{j=1}^{k} \frac{p_{\theta}\left(\mathbf{x}, \mathbf{z}^{(j)}\right)}{q\left(\mathbf{z}^{(j)}\right)}
$$

However, what we want to approximate is the marginal log-likelihood:
$$
\log \left(\sum_{z \in \mathcal{Z}} p_{\theta}(\mathbf{x}, \mathbf{z})\right)=\log \left(\sum_{z \in \mathcal{Z}} \frac{q(\mathbf{z})}{q(\mathbf{z})} p_{\theta}(\mathbf{x}, \mathbf{z})\right)=\log \left(\mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}\left[\frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})}\right]\right)
$$

It’s clear that

$$
\mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}\left[\log \left(\frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})}\right)\right] \neq \log \left(\mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}\left[\frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})}\right]\right)
$$

Idea: we can use Jensen Inequality:
$$
\log \left(\mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}[f(\mathbf{z})]\right)=\log \left(\sum_{\mathbf{z}} q(\mathbf{z}) f(\mathbf{z})\right) \geq \sum_{\mathbf{z}} q(\mathbf{z}) \log f(\mathbf{z})
$$

Thus 
$$
\log \left(\mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}\left[\frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})}\right]\right) \geq \mathbb{E}_{\mathbf{z} \sim q(\mathbf{z})}\left[\log \left(\frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})}\right)\right]
$$

This is also called Evidence Lower Bound (ELBO).


Now we came back to the unanswered question: what is a good choice of q(z)

Solution: Variational Inference: Optimize over the possible q’s to make
bound as tight as possible.

If $$q(\mathbf{z})=p(\mathbf{z} \mid \mathbf{x} ; \theta)$$ the bound becomes:

$$
\begin{aligned} \sum_{\mathbf{z}} p(\mathbf{z} | \mathbf{x} ; \theta) \log \frac{p(\mathbf{x}, \mathbf{z} ; \theta)}{p(\mathbf{z} | \mathbf{x} ; \theta)} &=\sum_{\mathbf{z}} p(\mathbf{z} | \mathbf{x} ; \theta) \log \frac{p(\mathbf{z} | \mathbf{x} ; \theta) p(\mathbf{x} ; \theta)}{p(\mathbf{z} | \mathbf{x} ; \theta)} \\ &=\sum_{\mathbf{z}} p(\mathbf{z} | \mathbf{x} ; \theta) \log p(\mathbf{x} ; \theta) \\ &=\log p(\mathbf{x} ; \theta) \sum_{\mathbf{z}} p(\mathbf{z} | \mathbf{x} ; \theta) \\ &=\log p(\mathbf{x} ; \theta) \end{aligned}
$$

However, In practice, the posterior $$p(\mathbf{z} \mid \mathbf{x} ; \theta)$$ is intractable to compute.

Suppose q(z) is any probability distribution over the hidden variables.
A little bit of algebra reveals

## Learning deep latent variable generative models
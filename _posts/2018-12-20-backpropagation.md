---
layout: post
title: "Backpropagation"
date: 2018-12-20
comments: true
---

## Introduction

Backpropagation is a powerful algorithm used to train modern neural networks. Most deep learning libraries implement backpropagation "under the hood," which forecloses the possibility of learning about its inner mechanics. Though backpropagation is a simple concept, it has many features that make it an interesting method to learn about. This article seeks to give practitioners a better intuition on what backpropagation is and how it can be applied to neural networks.

## Background

Neural networks---at the core---are composed of a sequence of differentiable functions. Each function or group of functions at a specific depth is commonly denoted as a layer---this provides an abstraction from which certain global operations can be performed. In this article, we will focus on simple feed-forward networks, although the concepts discussed can be extended to recurrent and convolutional architectures as well.

Global operations on neural networks include the _forward_ and _backward_ passes. Given an input $\mathbf{x} = [x^{(1)}, x^{(2)}, \cdots, x^{(n)}]$ and collection of parameters $\mathbf{\theta}$, the _forward_ pass computes latent activations and a scalar loss value. There are various types of loss functions, but they all use a deterministic method to quantify the divergence between the network's predicted values $\mathbf{\hat{y}}$ and the true values $\mathbf{y}$.

The training process modifies the parameters $\mathbf{\theta}$ to close the divergence as much as possible. During the _backward_ pass, $\mathbf{\theta}$ undergoes updates with the goal of minimizing the loss function $\ell$. The changes are performed by subtracting the derivative of the loss function with respect to $\mathbf{\theta}$ (formally, $\frac{d\ell}{d\mathbf{\theta}}$) multiplied with a non-zero learning rate $\alpha$. This process describes gradient descent---a popular optimization algorithm in machine learning.

To intuitively understand the backward pass, consider the definition of a derivative. Derivatives measure the change of a function's output with respect to a change to its parameters. Mathematically, the derivative can be expressed as follows:

$$f'(a) = \lim_{h \rightarrow 0} \frac{f(a+h)-f(a)}{h}$$

Rearranging the terms gives us the following equation:

$$f(a+h) \approx f(a) + f'(a)h$$

This linear approximation directly shows how a function's output is affected by the tangent line---intuitively, the rate of change---at $a$. For example, given the function $f(x,y) = xy$ where $x=2$ and $y=-6$, it is easy to show that $\frac{\partial f}{\partial x} = -6$. Using the formula above, we can infer that increasing $x$ by $h$ decreases the value of $f$ by $6h$, as per the negative gradient. This can be generalized---moving in the negative direction of the gradient minimizes the function.

## Chain Rule

Till now, we have only considered gradients of a _single_ function. Neural networks---as alluded to earlier---are composed of several functions. As a concrete example, consider the feed-forward network

$$
z_1 = f_1(\mathbf{x}, \mathbf{W}_1, b_1) \\
z_2 = f_2(z_1) \\
z_3 = f_3(z_2, \mathbf{W}_2, b_2) \\
\ell (z_3)
$$

where $z_i$ are latent activations, $\mathbf{W}_i$ are weights, and $b_i$ are biases. $\ell$ denotes the loss function and $f_i$ are intermediate transformations. The network can also be rewritten as the complicated function

$$\ell (f_3(f_2(f_1(\mathbf{x}, \mathbf{W}_1, b_1)), \mathbf{W}_2, b_2))$$

or with the composition operator $\circ$

$$\ell \circ f_3 \circ f_2 \circ f_1$$

Computing the gradients for the trainable parameters $\mathbf{W}_i$ and $b_i$ is non-trivial since they are used in functions that depend on other functions. For example, $\mathbf{W}_2$ is used as input to $f_3$, whose output is used as input to $\ell$.

We can use the _chain rule_ to compute these "nested gradients." Given an arbitrarily long composition of functions $f_1 \circ f_2 \circ \cdots \circ f_n$ with a single parameter $x$, the chain rule defines $\frac{df_1}{dx}$ as

$$
f'_{1 \cdots n}(x) = f'_1 (f_{2 \cdots n}(x))f'_2 (f_{3 \cdots n}(x))\cdots f'_n (x) \\
= \prod_{i=1}^{n} f'_i (f_{i+1 \cdots n}(x))
$$

where $f_{i \cdots j}$ is $f_i \circ \cdots \circ f_j$ given $j > i$. When applied to neural networks, the chain rule "unwinds" the layers until it reaches the parameter of interest. This is known as _backpropagation_---the recursive application of the chain rule to obtain gradients for trainable parameters in each layer. For our toy neural network, the gradient for $\mathbf{W}_2$ is calculated by

$$\frac{\partial \ell (z_3)}{\partial \mathbf{W}_2} = \frac{\partial \ell (z_3)}{\partial z_3} \frac{\partial z_3}{\partial \mathbf{W}_2}$$

Similarly, the gradient for $\mathbf{W}_1$ is calculated by

$$\frac{\partial \ell (z_3)}{\partial \mathbf{W}_1} = \frac{\partial \ell (z_3)}{\partial z_3} \frac{\partial z_3}{\partial z_2} \frac{\partial z_2}{\partial z_1} \frac{\partial z_1}{\partial \mathbf{W}_1}$$

In these examples, we use the $\partial$ operator because we are working with functions of multiple variables. The gradient of a particular variable is obtained by keeping the other variables constant.

## Gradient Computation

During backpropagation, there are a chain of matrix multiplications that must occur. For example, to find $\frac{\partial \ell (z_3)}{\partial \mathbf{x}}$, we use the following equation:

$$\frac{\partial \ell (z_3)}{\partial \mathbf{x}} = \frac{\partial \ell (z_3)}{\partial z_3} \frac{\partial z_3}{\partial z_2} \frac{\partial z_2}{\partial z_1} \frac{\partial z_1}{\partial \mathbf{x}}$$

There are four matrices that must be multiplied together to find the gradient of $\mathbf{x}$. This computation can either occur in the _forwards_ or _backwards_ direction. Note that these concepts are different than the _forwards_ and _backwards_ passes discussed above.

<img src="{{ site.baseurl }}/images/2018-12-20-1.png">

Let's first examine the _forwards_ computation. The complexity of multiplying the matrices $\frac{\partial \ell (z_3)}{\partial z_3}$ and $\frac{\partial z_3}{\partial z_2}$ is $O(1\vert z_3\vert \vert z_2\vert)$. Multiplying the intermediate matrix with $\frac{\partial z_2}{\partial z_1}$ is $O(1\vert z_2\vert \vert z_1\vert)$. Finally, multiplying the second intermediate matrix with $\frac{\partial z_1}{\partial \mathbf{x}}$ is $O(1\vert z_1\vert \vert \mathbf{x}\vert)$. This gives us a total complexity of $O(1\vert z_3\vert \vert z_2\vert) + O(1\vert z_2\vert \vert z_1\vert) + O(1\vert z_1\vert \vert \mathbf{x} \vert)$.

Similar analysis can be performed for the _backwards_ computation. The complexity of multiplying the matrices $\frac{\partial z_1}{\partial \mathbf{x}}$ and $\frac{\partial z_2}{\partial z_1}$ is $O(\vert z_2 \vert \vert z_1 \vert \vert \mathbf{x} \vert)$. Multiplying the intermediate matrix with $\frac{\partial z_3}{\partial z_2}$ is $O(\vert z_3 \vert \vert z_2 \vert \vert \mathbf{x} \vert)$. Finally, multiplying the second intermediate matrix with $\frac{\partial \ell (z_3)}{\partial z_3}$ is $O(1\vert z_3 \vert \vert \mathbf{x} \vert)$. This gives us a total complexity of $O(\vert z_2 \vert \vert z_1 \vert \vert \mathbf{x} \vert) + O(\vert z_3 \vert \vert z_2 \vert \vert \mathbf{x} \vert) + O(1\vert z_3 \vert \vert \mathbf{x} \vert)$.

$\mathbf{x}$ is involved in the matrix multiplication for each of the backwards computation steps in contrast to only one of the forward computation steps. In practice, $\mathbf{x}$ can contain hundreds of millions of elements, so the backwards computation is prohibitively expensive. Separating the training set into mini-batches certainly helps with the complexity, but the forwards computation is still far more efficient.

The forwards computation also enables us to cache intermediate results. For example, in a single backpropagation step, we can cache the intermediate gradient $\frac{\partial \ell (z_3)}{\partial z_1}$. When computing the gradients for the parameters $\mathbf{W}_1$ and $b_1$, we only need to perform matrix multiplications with $\frac{\partial z_1}{\partial \mathbf{W}_1}$ and $\frac{\partial z_1}{\partial b_1}$. The complete equations can be formulated as follows:

$$
\frac{\partial \ell (z_3)}{\partial \mathbf{W}_1} = \frac{\partial \ell (z_3)}{\partial z_1} \frac{\partial z_1}{\partial \mathbf{W}_1} \\
\frac{\partial \ell (z_3)}{\partial b_1} = \frac{\partial \ell (z_3)}{\partial z_1} \frac{\partial z_1}{\partial b_1}
$$

Caching is not particularly useful for the backwards computation since the scalar loss is not the starting point.

## Conclusion

In this article, we covered backpropagation---a simple, yet powerful method used to train neural networks. Backpropagation computes the gradients necessary to adjust the weights in each layer. Because neural networks can be thought of as gigantic nested functions, gradients are obtained through recursive applications of the chain rule. Further, the direction in which the chain rule calculation flows also has several implications; the forwards computation is not only more efficient, but also exploits caching in modern systems.

## References

[1] [https://en.wikipedia.org/wiki/Derivative](https://en.wikipedia.org/wiki/Derivative)

[2] [http://cs231n.github.io/optimization-2/](http://cs231n.github.io/optimization-2/)

[3] [https://en.wikipedia.org/wiki/Gradient_descent](https://en.wikipedia.org/wiki/Gradient_descent)

[4] [https://en.wikipedia.org/wiki/Chain_rule](https://en.wikipedia.org/wiki/Chain_rule)

[5] [http://www.philkr.net/cs342](http://www.philkr.net/cs342)
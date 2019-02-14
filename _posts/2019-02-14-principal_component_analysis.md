---
layout: post
title: "Principal Component Analysis"
date: 2019-02-14
comments: true
---

## Introduction

Principal Component Analysis (PCA) is an unsupervised algorithm used to represent high dimensionality data in lower dimensions. Given a set of observations $x^{(1)}, \cdots, x^{(n)}$ where $x^{(i)} \in \mathbb{R}^d$, PCA linearly projects the data onto a space with dimensionality $d'$ where $d' < d$. The intuition is that in a high-dimensional space, there may be several correlated variables, but only a subset of those variables---known as _principal components_---capture the essence of the data. Fewer dimensions can be used to approximate the original data without losing too much information. PCA has several important applications in machine learning, computer vision, bioinformatics, and quantitative finance. This article is a deep-dive into the derivation and properties behind PCA.

- [Concept Review](#concept-review)
    - [Covariance](#covariance)
    - [Lagrange Multiplier](#lagrange-multiplier)
    - [Eigenvalues and Eigenvectors](#eigenvalues-and-eigenvectors)
- [Maximizing Variance](#maximizing-variance)
- [Principal Components](#principal-components)
- [Properties of Eigenvectors](#properties-of-eigenvectors)
- [Conclusion](#conclusion)

## Concept Review

Before getting into technical details, it might be worth to brush up on the following concepts from statistics, optimization, and linear algebra. If you are already familiar with these concepts, feel free to skip ahead.

### Covariance

The *covariance* of two random variables $X$ and $Y$ describes how linearly correlated they are. Formally, this can be shown as 

$$\mathrm{Cov}(X,Y) = \mathbb{E}[(X-\mathbb{E}[X])(Y-\mathbb{E}[Y])]$$

It is often useful to describe the linear relationship between multiple random variables. The *covariance matrix* $\Sigma$ of a vector $\mathbf{v} \in \mathbb{R}^n$ that holds $n$ random variables is a $n \times n$ matrix where $\Sigma_{ij} = \mathrm{Cov}(\mathbf{v}_i, \mathbf{v}_j)$. The diagonal of the covariance matrix is simply the variance of each random variable $\mathbf{v}_i$ since $\mathrm{Var}(\mathbf{v}_i) = \mathrm{Cov}(\mathbf{v}_i, \mathbf{v}_i)$.

### Lagrange Multiplier

*Lagrange multipliers* describes an optimization method to find the local minima or maxima of a function under specific constraints. For single constraint problems, we define the Lagrange function

$$\mathcal{L}(\mathbf{\theta}, \lambda) = f(\mathbf{\theta}) - \lambda \cdot g(\mathbf{\theta})$$

where $f$ is the function to be optimized under the constraint $g$ using the paramters $\mathbf{\theta}$. The gradients for $f$ and $g$ are required to find the critical points of $f$. In addition, $\lambda$ is the Lagrange multiplier, which is merely a constant used to scale the gradients. If you are confused, don't worry---it is only used in one part of the derivation. 

### Eigenvalues and Eigenvectors

*Eigenvectors* are special non-zero vectors that change only by a scalar factor when linear transformations are applied to them. For example, given the linear transformation $\mathbf{A}$, the eigenvector $\mathbf{v}$ satisfies the equation

$$\mathbf{A}\mathbf{v} = \lambda \mathbf{v}$$

The scalar $\lambda$ is also known as the *eigenvalue*. Note that Lagrange multipliers and eigenvalues are distinct concepts although $\lambda$ is used to represent both of them.

## Maximizing Variance

PCA finds a subspace for the space the data originally lies in, but it is non-trivial to determine what the _optimal_ subspace is. Due to the [curse of dimensionality](https://en.wikipedia.org/wiki/Curse_of_dimensionality), we cannot use a naive brute-force strategy to search for the subspace. This problem can be ameliorated by framing the search as an optimization problem---the optimal subspace must ultimately minimize or maximize some target function.

To understand the problem better, consider the dataset $x^{(i)}, \cdots, x^{(n)}$ with linearly correlated features $x_1$ and $x_2$. The points can be thought of as lying on a latent diagonal vector $\mathbf{u}$ with some additional noise.

<img src="{{ site.baseurl }}/images/2019-02-14-1.png">

Each point $x^{(i)}$ is two-dimensional---it requires two points to represent its position. However, if the points were [projected](https://en.wikipedia.org/wiki/Projection_(linear_algebra)) onto $\mathbf{u}$, they would only require one coordinate to represent its position. In two dimensions, the projection $x^{(i)} \cdot \mathbf{u}$ would look like this:

<img src="{{ site.baseurl }}/images/2019-02-14-2.png">

The projection provides a lower-dimensional coordinate system to represent the points while maintaining the integrity of the dataset. The points cluster towards the middle and are sparse towards the ends in both representations. However, the projection is still an approximation of the original dataset---some information is lost, but the core components remain.

In this particular example, $x^{(i)} \cdot \mathbf{u}$ represents the distance from the origin while $(x^{(i)} \cdot \mathbf{u})\mathbf{u}$ gives the coordinate of the point in two dimensions. Think about how this formula relates to the image shown above. Through a least squares objective, we can minimize the squared error between the original point $x^{(i)}$ and the reconstructed point $(x^{(i)} \cdot \mathbf{u})\mathbf{u}$:

$$\ell (\mathbf{u}) = \frac{1}{n} \sum_{i=1}^n (x^{(i)}-(x^{(i)} \cdot \mathbf{u})\mathbf{u})^2$$

Minimizing the reconstruction error turns out to be equivalent to maximizing the variance of $\mathbf{u}$ (see [this proof](https://www.stat.cmu.edu/~cshalizi/uADA/12/lectures/ch18.pdf) for additional details), given the following conditions:
- *Each feature is centered.* The mean of each feature, in this case $x_1$ and $x_2$, should be $0$. This eliminates variability in the units the featues were measured in. More importantly, it simplifies the calculation of the covariance matrix $\Sigma$ required for PCA, which will be discussed in the next section.
- $\mathbf{u}$ *is a unit vector.* By definition, unit vectors have a magnitude of $1$ ($\| \| \mathbf{u} \| \| = 1$). The variance of $\mathbf{u}$ can get arbitrarily large as long as its magnitude increases. To ensure the possibility of one unique solution, we can constrain the magnitude of $\mathbf{u}$. This is permissible as we only really care about the direction of $\mathbf{u}$.

Therefore, the optimal subspace is simply the one with the most variance. This makes sense intuitively because variance is strongly linked to information. For example, consider a sentiment analysis task supervised on a Twitter dataset. The features given to the model include (a) "character count" and (b) "number of positive words." Feature (a) does not vary much because Twitter imposes a character limit on tweets. However, each tweet differs in its content, so feature (b) would vary quite a bit. The model will probably use feature (b) more when establishing classification boundaries as it offers more information than feature (a).

## Principal Components

Now that we have built the intuition that PCA finds reduced subspaces of maximum variance, we can shift our focus to the _principal components_ (PC) that form these subspaces. The fist PC finds the direction of maximum variance, the second PC finds the maximum variance in a direction [orthogonal](https://en.wikipedia.org/wiki/Orthogonality) to the first, the third PC finds the maximum variance in a direction orthogonal to the second, and so on. The first two PCs of our example dataset are shown below:

<img src="{{ site.baseurl }}/images/2019-02-14-3.png">

It is clear that the points vary the most _across_ the diagonal, then vary the most _off of_ the diagonal; these are the exact trends captured by the two principal components. In low-dimensional datasets such as this one, it is trivial to visualize the directions of maximum variance, but it is much more difficult to do so for high-dimensional datasets. We will now walk through the derivation of PCA to automatically find these principal components.

Assume we are given the dataset $x^{(1)}, \cdots, x^{(n)}$ where $x^{(i)} \in \mathbb{R}^d$, generalizing the previous example to $d$ variables. If the vectors $x^{(i)}$ are stacked row-wise into a matrix $\mathbf{X}$, the rows represent observations while the columns represent variables. For simplicity, we will derive the equations to find the first principal component $\mathbf{u}$.

Most machine learning algorithms begin with preprocessing---we must center the data, computing 

$$\mathbf{\mu} = \frac{1}{n} \sum_{i=1}^n x^{(i)}$$

then replacing each $x^{(i)}$ with $x^{(i)} - \mathbf{\mu}$. Note that "centering" is different than "normalizing," which performs the additional step of rescaling each variable to have unit variance. Rescaling is not explicitly required, but it can be implemented if the given task benefits from it.

Next, we must maximize the variance of the projection $x^{(i)} \cdot \mathbf{u}$. We can simplify the formula using various matrix rules

$$
\begin{equation}
\begin{split}
\operatorname*{argmax}_{\mathbf{u}} \mathrm{Var}(x^{(i)} \cdot \mathbf{u}) & = \frac{1}{n} \sum_{i=1}^n (x^{(i)} \cdot \mathbf{u})^2 \\
& = \frac{1}{n} \sum_{i=1}^n (x^{(i)} \cdot \mathbf{u})(x^{(i)} \cdot \mathbf{u}) \\
& = \frac{1}{n} (\mathbf{X} \mathbf{u})^T(\mathbf{X} \mathbf{u}) \\
& = \frac{1}{n} \mathbf{u}^T \mathbf{X}^T \mathbf{X}^T \mathbf{u} \\
& = \mathbf{u}^T (\frac{1}{n} \mathbf{X}^T \mathbf{X}) \mathbf{u} \\
& = \mathbf{u}^T \mathbf{\Sigma} \mathbf{u}
\end{split}
\end{equation}
$$

where $\mathbf{\Sigma}$ is the covariance matrix of $\mathbf{X}$. Why does this work? $\mathbb{E}[\mathbf{X}] = 0$ as per the centering of the data, so the covariance matrix computation reduces to $\mathbb{E}[\mathbf{X}^T \mathbf{X}]$.

Now, we must choose a unit vector $\mathbf{u}$ to maximize the projected variance of $\mathbf{u}^T \mathbf{\Sigma} \mathbf{u}$. The unit vector $\mathbf{u}$ is the constraint---$\| \| \mathbf{u} \| \| = 1$ or $\mathbf{u}^T \mathbf{u} = 1$. We can use Lagrange multipliers to solve this constrained optimization problem

$$\mathcal{L}(\mathbf{u}, \lambda) = \mathbf{u}^T \mathbf{\Sigma} \mathbf{u} - \lambda (\mathbf{u}^T \mathbf{u} - 1)$$

where $\mathbf{u}^T \mathbf{\Sigma} \mathbf{u}$ is the target function and $\mathbf{u}^T \mathbf{u} - 1$ is the constraint function. The solution exists when the partial derivatives of $\mathcal{L}$ are equal to $0$. Taking the gradient of $\mathcal{L}$ with respect to $\mathbf{u}$ and setting it equal to $0$, we obtain the following:

$$
\nabla_{\mathbf{u}} \mathcal{L}(\mathbf{u}, \lambda) = 2 \mathbf{\Sigma}\mathbf{u} - \lambda \cdot 2 \mathbf{u} = 0 \\
\mathbf{\Sigma} \mathbf{u} = \lambda \mathbf{u}
$$

This implies that the principal component $\mathbf{u}$ is an eigenvector of the covariance matrix $\mathbf{\Sigma}$ (more on this in the next section). In fact, it turns out that the eigenvectors of $\mathbf{\Sigma}$ are all principal components. If the $d$ eigenvalues of $\mathbf{\Sigma}$ are sorted in increasing order,

$$\lambda_1 \gt \lambda_2 \gt \cdots \gt \lambda_d$$

their corresponding eigenvectors are the top $d$ principal components

$$\mathbf{u}_1, \mathbf{u}_2, \cdots , \mathbf{u}_d$$

where $\mathbf{u}_1$ and $\mathbf{u}_d$ are the directions of maximum and minimum variance, respectively.

## Properties of Eigenvectors

It might come as a surprise that eigenvalues and eigenvectors get mixed into the derivation, but there is an intuitive mathematical explanation for this.

The covariance matrix $\mathbf{\Sigma}$ explains the correlation between multiple random variables. To understand this graphically, consider the following multivariate Gaussian distributions controlled by the random variables $X$ and $Y$. The distributions and their respective covariance matrices are shown below:

<img src="{{ site.baseurl }}/images/2019-02-14-4.png">

Notice that $\mathbf{\Sigma}$ encodes geometric properties of the multivariate Gaussian; in particular, the variance controls the spread while the covariance controls the orientation. To see the effect of $\mathbf{\Sigma}$ as a linear transformation, let's see what happens when it is repeatedly multipled with an arbitrary $\mathbf{u}$.

The following plots are generated with this simple procedure:
1. Fix arbitrary $\mathbf{u}$
2. Let $\mathbf{u}' = \mathbf{\Sigma} \mathbf{u}$
3. Replace $\mathbf{u}$ with $\mathbf{u}'$
4. Repeat steps 1-3 for $k$ iterations

<img src="{{ site.baseurl }}/images/2019-02-14-5.png">

It is clear from these plots that $\mathbf{\Sigma}$ has interesting properties when interpreted as a linear transformation. In particular, after each multiplication with $\mathbf{\Sigma}$, the magnitude of $\mathbf{u}$ increases and the slope of $\mathbf{u}$ converges to the direction of maximum variance. This describes an iterative process to find the first principal component---for any $\mathbf{u}$, we can repeatedly perform $\mathbf{\Sigma} \mathbf{u}$ utnil the slope of $\mathbf{u}$ converges.

In other words, the goal is to find $\mathbf{u}$ such that multiplying it with $\mathbf{\Sigma}$ only affects its magnitude, not its direction. This implies $\mathbf{u}$ is an eigenvector of $\mathbf{\Sigma}$ as the linear transformation $\mathbf{\Sigma} \mathbf{u}$ only scales $\mathbf{u}$ by a scalar $\lambda$---the eigenvalue---but does not change its direction. Geometrically, eigenvectosr are directions "stretched" by a linear transformation, so it is intuitive that the eigenvectors of $\mathbf{\Sigma}$ fulfill the properties required for $\mathbf{u}$.

Further, because $\mathbf{\Sigma}$ is a symmetric matrix with dimensionality $d \times d$, it can be shown that the eigenvectors corresponding to unique eigenvalues are orthogonal, fulfilling the property that principal components are orthogonal. This geometric consequence is intuitive---if we visualize vectors as capturing an elliptical "cloud of variance," only orthogonal vectors capture the maximum variance.

<img src="{{ site.baseurl }}/images/2019-02-14-6.png">

In summary, projecting a (centered) $d$-dimensional matrix $\mathbf{X}$ into a $k$-dimensional subspace ($k \lt d$) requires taking the top $k$ eigenvectors of the covariance matrix of $\mathbf{X}$. In Numpy, this can be implemented in a few lines of code:

```python
import numpy as np

x = ... # number of observations
m = ... # dimensionality of original dataset
k = ... # dimensionality of compressed dataset

X = np.random.rand(x, k)
X -= np.mean(X, axis=0)
cov = np.dot(X.T, X)
evals, evecs = np.linalg.eig(cov)
components = evecs[:, :k]
```

## Conclusion

PCA finds an underlying subspace for high-dimensional spaces with no prior knowledge of their shapes or structures. It can be mathematically and intuitively shown that the variance of this subspace aptly describes the correlations between variables. Using simple eigenvalue/eigenvector calculations, we can derive orthogonal directions of maximum variance, commonly known as principal components. Each component represents a unit of information; collectively, they form the crux of the original dataset. Though PCA has its roots in statistical machine learning, it has applications ranging across the board, from bioinformatics to quantitative finance.

## References

[1] [https://en.wikipedia.org/wiki/Principal_component_analysis](https://en.wikipedia.org/wiki/Principal_component_analysis)

[2] [https://en.wikipedia.org/wiki/Covariance_matrix](https://en.wikipedia.org/wiki/Covariance_matrix)

[3] [https://en.wikipedia.org/wiki/Lagrange_multiplier](https://en.wikipedia.org/wiki/Lagrange_multiplier)

[4] [Pattern Recognition and Machine Learning, Chapter 12](https://www.microsoft.com/en-us/research/publication/pattern-recognition-machine-learning/)

[5] [http://www.maths.manchester.ac.uk/~mkt/MT3732%20(MVA)/Notes/MVA_Section2.pdf](http://www.maths.manchester.ac.uk/~mkt/MT3732%20(MVA)/Notes/MVA_Section2.pdf)

[6] [https://www.stat.cmu.edu/~cshalizi/uADA/12/lectures/ch18.pdf](https://www.stat.cmu.edu/~cshalizi/uADA/12/lectures/ch18.pdf)

[7] [http://www.math.harvard.edu/archive/21a_spring_09/PDF/11-08-Lagrange-Multipliers.pdf](http://www.math.harvard.edu/archive/21a_spring_09/PDF/11-08-Lagrange-Multipliers.pdf)

[8] [https://math.stackexchange.com/questions/82467/eigenvectors-of-real-symmetric-matrices-are-orthogonal](https://math.stackexchange.com/questions/82467/eigenvectors-of-real-symmetric-matrices-are-orthogonal)

[9] [http://www.visiondummy.com/2014/04/geometric-interpretation-covariance-matrix/](http://www.visiondummy.com/2014/04/geometric-interpretation-covariance-matrix/)
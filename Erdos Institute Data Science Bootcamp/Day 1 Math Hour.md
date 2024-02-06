Framework for parametric supervised machine learning.
Data is $n$ points in $\mathbb{R}^p$, so it is represented as an $n\times p$ matrix $X$. The framework for parametric supervised machine learning is though of as a map $f:\mathbb{R}^p\to\mathbb{R}^k$ so the output is called $Y$ which is $n\times k$. We are trying to find the "true function" $f\in\mathcal{F}$ where $\mathcal{F}$ is a family of functions (such as linear functions, logistic function, or neural networks). The *parametric* part comes form the fact that this class of function $\mathcal{F}$ should be specified by some set of parameters.

For example, $f_\theta(\mathbb{x}) = \theta_0 + \mathbf{\theta}_{1,\ldots,n}\cdot \mathbf{x}$ would be in the class of linear models since there is **linearity in the parameters**. We will specify a "loss function" which quantifies how well a given $f_\theta$ represents the data.

Two common loss functions:
1. Maximum likelihood estimation (MLE)
2. Maximum a posteriori estimation (MAP)

# Maximum likelihood estimation
Example: Let's say we have a weighted coin where the probability of heads is some parameter $\theta$ which we are trying to estimate. If we flip the coin three times and the outcomes are HHT, then we may estimate $\theta=2/3$. However, if we weren't told the coin was weighted and we had the same outcomes we would still say our $\theta=1/2$, this is getting to the idea of the difference between MLE and MAP.

Now in general, the likelihood of a coin weighted $\theta$ producing HHT is $L(\theta)=\theta^2(1-\theta)$. The maximum value of this function over $[0,1]$ can be found using differential Calculus:
$$\frac{dL}{d\theta} = -\theta^2 + 2\theta(1-\theta)=\theta(2-3\theta)=0$$

We are getting a minimum at $\theta=0$ and a maximum at $\theta=2/3$.

Now in general if we have $m$ heads and $n-m$ tails, instead of maximizing the likelihood we will maximize the log likelihood:

$$L(\theta)=\theta^m(1-\theta)^{n-m}$$

The log likelihood is as follows:

$$\log(L)=m\log(\theta) + (n-m)\log(1-\theta)$$

and thus the derivative is easier to compute:

$$\frac{d\log(L)}{d\theta}=\frac{m}{\theta}-\frac{n-m}{1-\theta}.$$

# Maximum a posteriori estimation
Real world coins have $\theta\approx1/2$, if we flip the coin five times and get 4 heads and 1 tails what would you .
In Bayesian statistics we have some prior information (e.g., most coins are not biased). You can think of MLE as MAP where the prior distribution is uniform.
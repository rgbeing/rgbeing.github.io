---
layout: post
title: "Correlation Can Be Used As Distance in Clustering"
author: "Dojun Hwang"
comments: true
---

# How This Works

From the previous example, we found that clustering using correlations as a distance metric is *somewhat* effective in clustering by patterns. 
But, does it make sense to use correlation as a distance metric? Can we justify our decision?
Actually, the $1-Corr$ itself is not a distance metric. This is because it does not meet the triangle inequality.

### Example: Correlation does not meet the triangle inequality
Suppose that
$$\begin{aligned} x_1 &= [4, 9, 2] \\ x_2 &= [2,4,7] \\ x_3 &= [0,1,4] \end{aligned}$$

Then, the *population* correlations are
$$\rho = \begin{bmatrix}1 &  &  \\ -0.3857 & 1 & \\ -0.5329 & 0.9862 & 1 \end{bmatrix}$$
so that the distance matrix is
$$d = \begin{bmatrix}0 &  &  \\ 1.3857 & 1 & \\ 1.5329 & 0.0138 & 1 \end{bmatrix}$$
Here, $d(x_1, x_3) = 1.5329 > d(x_1, x_2) + d(x_2, x_3) = 1.3995$. Thus the triangle inquality does not hold here.
Hence, using correlation as a distance metric is not applicable for every cases, and may lead to the somewhat strange clustering result. 
However, we can show that correlation can be a distance metric in a different perspective. 
**Actually, $1 - Corr(x, y)$ is a distance between *standardized* $x$ and *standardized* $y$.** 

### (1 - Correlation) is a squared distance between row-wise standardized data

**Theorem**. Suppose we are given sequence of data $x = (x_1, \cdots, x_k)$ and $y = (y_1, \cdots, y_k)$. 
Suppose these data are standardized ones, so that
$$E(x) = \dfrac{1}{k}\sum_i x_i = E(y) = \dfrac{1}{k}\sum_i y_i = 0,$$ and $$Var(x) = \dfrac{1}{k}\sum_i [x_i - E(x)]^2 = Var(y) = \dfrac{1}{k}\sum_i [y_i - E(x)]^2 = 1.$$
(Note that the variance is population variance, not sample.) 
Then, $$||x - y||^2_2 \propto 1 - Corr(x, y),$$ where $Corr(x, y)$ is the Pearson correlation between $x$ and $y$.

**Proof**. As $E(x) = E(y) = 0$ and $Var(x) = Var(y) = 1$, it is also that $E(x^2) = E(y^2) = 1$. Furthermore,
$$\begin{aligned} Corr(x, y) &= \dfrac{Cov(x, y)}{\sqrt{Var(x) Var(y)}}\\ &= Cov(x, y) \\  &=\dfrac{1}{k} \sum_i (x_i - E(x))(y_i - E(y)) \\ &= \dfrac{1}{k}\sum_i x_i y_i \\ &= E(xy). \end{aligned}$$ Then,
$$\begin{aligned} ||x - y||^2_2 &= \sum_i (x_i - y_i)^2 \\ &= \sum_i(x_i^2 + y_i^2 - 2x_iy_i) \\ &= kE(x^2) + kE(y^2) - 2kE(xy) \\ &= 2k[1-Corr(x, y)]. \end{aligned}$$ 
Hence $||x - y||^2_2 \propto 1 - Corr(x, y)$.

This theorem implies that, **using (1 - correlation) between two objects as a distance is equivalent to use the distance between *row-wise standardized* data.** 
Hence clustering data using correlation as similarity measure may make sense.

However, we must focus on that it is equivalent to distance between *row-wise standardized* data; but standardization is usually done on column-wise manner. This is because, generally, there is no point in standardizing observations in different columns. Imagine that, there is data looks like this:

| Age | Income | Number of children |
| --- | ------ | ------------------ |
| 48  | 100000 | 2                  |
Is standardizing this data by row make sense? It does not, as each column delivers very different features of each object. 
However, **time-series data** is special, as each column is entangled with others. We may be able to predict the outcome at $t_5$ using the numbers in column $t_1, t_2, t_3$, and $t_4$. 
Therefore it does make sense to use correlation as a similarity measure in the case of time-series data.

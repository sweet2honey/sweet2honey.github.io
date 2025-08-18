---
title: 白话机器学习的数学 - 2 Regression
mathjax: true
toc: true
categories:
  - machine-learning
tags:
  - ml
date: 2025-08-15 14:53:23
description:
	回归入门：最小二乘法，多项式回归和随机梯度下降。
---

# 最小二乘法
假设有二元函数：
$$
f_{\theta}(x)=\theta_{0}+\theta_{1}x
$$

我们将训练数据中的广告费代入 $f_{\theta}(x)$  ，把得到的点击量与训练数据中的点击量相比较，然后找出使二者的差最小的 $\theta$，这种问题叫做**最优化问题**。

误差之和可以用表达式表示，称为**目标函数**：

$$
E(\theta) = \frac{1}{2}\sum_{i=1}^{n}(y^{(i)}-f_{\theta}(x^{(i)}))^2
$$
> 1. 这里的 $x^{（i）}$ 是指第 $i$ 个训练数据，不是次幂。
> 2. 为了避免误差相互抵消，所以不能简单把误差累加；这里不用绝对值用平方，是因为后面要微分，平方的微分比绝对值简单。

# 最速下降法

微分是计算变化的快慢程度时使用的方法。

对于一个函数 $f(x)$ 来说，只要向与导数的符号相反的方向移动 $x$，$f(x)$ 就会自然而然地沿着**极小值**的方向前进了。

这也被称为**最速下降法(Steepest descent）** 或 **梯度下降（Gradient descent）** 法：
$$
x := x - \eta\frac{d}{dx}f(x)
$$

$\eta$ 是称为学习率的正常数，影响收敛速度，甚至可能导致计算结果无法收敛。

---

$f_{\theta}(x)$ 是关于 $\theta_0$，$\theta_1$ 的双变量函数，所以要用偏微分：
$$
\theta_0 := \theta_0 - \eta\frac{\partial E}{\partial\theta_0}
$$
$$
\theta_1 := \theta_1 - \eta\frac{\partial E}{\partial\theta_1}
$$

发现 $E(\theta)$ 不是对 $\theta$ 的直接函数，可以考虑使用**复合函数**的微分法则，假设：
$$
E=E(\theta)
$$
$$
v=f_\theta(x)
$$
则有：
$$
\frac{\partial E}{\partial \theta _{0}} = \frac{\partial E}{\partial v} \cdot \frac{\partial v}{\partial \theta _{0}}
$$

---

$$
\begin{align*}
	\frac{\partial E}{\partial v}
	&= \frac{\partial}{\partial v} 
	\left({ \frac{1}{2} \sum_{i=1}^{n} \left(y^{(i)} - v\right)^2}\right) && \text{代入$E_\theta$}
	\\\\
	&={ \frac{1}{2} \sum _{i=1}^{n} \left(
	{ \frac{\partial u}{\partial v} \left(y^{(i)} - v\right)^2} \right)} && \text{微分分配律}
	\\\\
	  & ={\frac{1}{2} \sum _{i=1}^{n} \left({\frac{\partial u}{\partial v} \left({y^{(i)}}^2 -2y^{(i)}v +v^2\right)} \right)} && \text{展开平方} 
	\\\\
	  & ={\frac{1}{2} \sum _{i=1}^{n} \left(-2y^{(i)} + 2v \right) }                                                          && \text{对$v$求导}   
	\\\\
	  & ={\sum _{i=1}^{n} \left(v - y^{(i)}\right)}                                                                           && \text{常量提取} 
\end{align*}
$$

---

# 多项式回归


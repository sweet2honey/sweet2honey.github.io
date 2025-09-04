---
title: Rotating a point about center
mathjax: false
toc: true
categories:
  - Geometry
tags:
  - math
  - geometry
date: 2025-09-03 10:42:19
description:
---

昨天跟 Claude 搏斗了好久，原来是我想错了，速速认错。

---

一个点 $P(x,y)$ 绕 $C(x_0,y_0)$，**逆时针**旋转 $\theta$，结果得到 $P'(x',y')$ 满足：
$$
P'=C+R(P-C)
$$

其中 $R$ 是二维旋转矩阵：
$$
\begin{bmatrix}
\cos\theta && -\sin\theta \\
\sin\theta && \cos\theta
\end{bmatrix}
$$

或者直接展开得：
$$
x'=x_0 + (x-x_0)\cos\theta - (y-y_0)\sin\theta \\
y'=y_0 + (x-x_0)\sin\theta + (y-y_0)\cos\theta
$$

我被“坑”的地方是对旋转矩阵的定义，或者说使用场景没有建立正确的认知，旋转矩阵作用的坐标，**一定要是相对旋转中心的**。
在这个 case 中，旋转中心是 $C$，所以要先通过 $(P-C)$ 把 $P$ 转换到旋转中心（点 $C$）的坐标系下，计算完之后，重新恢复到原来的世界坐标系下。
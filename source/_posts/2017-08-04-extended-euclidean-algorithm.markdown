---
layout: post
title: "扩展欧几里得算法（Extended Euclidean algorithm）"
date: 2017-08-04 09:07:10 +0800
comments: true
categories: [algorithm]
---
欧几里得算法几乎算是我们最早接触到的算法，小学生就开始使用了。其实我更偏向于称之为辗转相除法，因为数学在中国的起源真的很早，然而文化背景的原因，这些宝贵的知识一直当作奇技淫巧。今天这里并不是要重复辗转相除法，而是讲它的一个有趣的变型：扩展辗转相除法。

<!--more-->

## 重温辗转相除法（Eculdiean algorithm）的过程

__例__： 求 $888$ 和 $54$ 的最大公约数 $gcd(888,54)$ 。

__解__：

$$
\begin{align}
888 &= 16 \times 54 + 24 & \text{(1)}\\\
54 &= 2 \times 24 + 6 & \text{(2)} \\\
24 &= 4 \times 6 + 0 & \text{(3)}
\end{align}
$$

倒数第二步 $(2)$ 的余数就是最后的结果，即 $gcd(888,54) = 6$

## 扩展辗转相除法（Extended Eculdiean algorithm）
在求解上例的过程中，我们总可以找到这样一组整数 $(x,y)$ 使得下式[^1]成立。

$$ ax + by = gcd(a,b) $$

具体作法：将 $gcd(a,b)$ 的求解过程反过来，每一步等式左边代入前一步，直到第一步。拿上例来说，过程如下：

$$
\begin{align}
6 & = 54 + (-2) \times 24 & &  (\text{变型}(2)) & \text{(4)} \\\
6 & = 54 + (-2) \times (888 + (-16)\times 54) & & \text{(用(1)变型代入 24)} & \text{(5)} \\\
6 &= (-2)\times 888 + 33 \times 54 & & \text{(合并同类项)} & \text{(6)}
\end{align}
$$

求得 $(x,y) = (-2, 33)$。到这里，大多数讲扩展辗转相除法的内容就停止了，其实这离真正可以应用和实现还差一步。

我们来看看一般形式，假设我们用了 $n$ 步迭代，应用辗转相除法求解出最大公约数 $gcd(a,b)$，即：

$$
\begin{align}
a_{n-1} &=  d_{n-1}\times b_{n-1} + c_{n-1} & \text{(7)} \\\
a_{n-2} &=  d_{n-1}\times b_{n-2} + c_{n-2} & \text{(8)} \\\
\cdots \\\
a_{i} &=  d_{i}\times b_{i} + c_{i} & \text{(9)} \\\
\cdots \\\
a_{0} &=  d_{0}\times b_{0} + c_{0} & \text{(10)} 
\end{align}
$$

容易发现 $b_{i+1} = a_{i}, c_{i+1} = b_{i}$。
假设最后一步是$b_{0} = 0$，那么 $gcd(a,b)=a_{0}$ 同时，我们可以假设在第 $i$ 步有：

$$
\begin{align}
& a_{i} x_{i} + b_{i} y_{i} = a_{0} & \text{(11)}
\end{align}
$$

可以通过将 $i+1$ 步的变型（$c_{i+1} = b_{i}$）代入 $b_{i}$ ：

$$
\begin{align}
& b_{i+1} x_{i} + (a_{i+1} + ( - d_{i+1})\times b_{i+1}) y_{i} = a_{0} & \text{(12)} \\\
& a_{i+1} y_{i} +  b_{i+1}(x_{i} - d_{i+1} y_{i} ) = a_{0} & \text{(13)}
\end{align}
$$

事实上，$(13)$ 符合 $(11)$ 的形式（$i>0$），可以比较系数得出：

$$
\begin{cases}
x_{i+1} = y_{i} \\\
y_{i+1} = x_{i} - d_{i+1} y_{i}
\end{cases}
$$

对于 $i=0$ 的边界情况，我们可以直接用 $\text{(11)}$ 得出：


$$
\begin{cases}
x_{0} = 1 \\\
y_{0} = 0
\end{cases}
$$

需要注意的是 $d_{i} = [a_{i}/b_{i}]$ ，$c_{i} = a_{i} \mod b_{i}$

## 应用及代码实现（Python）
扩展辗转相除法是求解模反元素（Modular multiplicative inverse）的方法之一，求解模反元素是 RSA 加密算法[^2]的主要步骤之一：

$$
a \equiv 1 \pmod{b}
$$


或者表达为：

$$
a x + by = 1
$$

实现扩展辗转相除法的Python 代码如下：

<script src="https://gist.github.com/skychan/8c21d22f7698af718f93d780a933a612.js"></script>

[^1]: [裴蜀等式](https://en.wikipedia.org/wiki/B%C3%A9zout%27s_identity)
[^2]: [RSA 加密算法](https://en.wikipedia.org/wiki/RSA_(cryptosystem))
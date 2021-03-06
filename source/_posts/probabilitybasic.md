title: 概率论基础
date: 2018-12-05 16:43:21
tags: Math
mathjax: true
---

> 概率论基础概念
<!-- more -->
# 条件概率

* 定义

设有两个事件$A,B$，并且$P(B) \neq 0$，在给定$B$发生的条件下，$A$的条件概率，为$P(A|B)$，定义为：

$${P(A|B) = \frac{P(AB)}{P(B)}, P(B) \gt 0}$$

* 乘法公式

$${P(AB) = P(B)P(A|B)}$$

# 全概率公式与贝叶斯公式

* 全概率公式

$若B_1,B_2,B_3,...,B_n是S的划分$，则

$${P(A) = \sum_{j=1}^{n}P(B_j)P(A|B_j)}$$

* 贝叶斯公式

在全概率公式的假定下：

$$P(B|A) = \frac{P(AB_i)}{P(A)} = \frac{P(B_i)P(A|B_i)}{\sum_{j}P(B_j)P(A|B_j)}$$

关于贝叶斯公式的理解：

1. 如果把事件$A$看成结果，事件$B_1, B_2, ...$看成导致这个结果的可能的原因，则可以形象的把全概率公式看成`由原因推结果`；而贝叶斯公式则恰好相反，其作用在于`由结果推原因`。
2. $A$是先验概率，$B$是后验概率。贝叶斯公式描述了先验概率已知的情况下，后验概率对先验概率的修正。

---

# 离散随机变量及概率分布

* 概率函数

设$X$为离散型随机变量，其全部可能值为$\{a1, a2, \cdots\}$，$X$的概率函数：
$${p_i = P(X = a_i), i = 1,2,\cdots}$$

* 分布函数

$${P(X \le x) = F(x), -\infty < x < +\infty}$$

离散型随机变量的分布函数为阶梯函数，$F(x)$在$x = a_i, (i = 1,2,...)$处跳跃，跳跃值为$p_k = P\{X = a_k\}$

* 数学期望

$$E(x) = \sum_{i=1}^{\infty}{x_i}{p_i}$$

* 方差

$$D(X)=E\{[X - E(X)]^2\}=E(X^2)-[E(X)]^2$$

## 1.伯努利分布

* 概率函数

$$ P(x) =
\begin{cases}
1 - p, & x = 0 \\
p,     & x = 1
\end{cases}
$$

* 期望

$$E(x) = 1 \times p + 0 \times (1 - p) = p$$

* 方差

$$D(x) = E(x^2) - [E(x)]^2 = p - p^2 = p(1 - p)$$

* 两点分布

## 2.二项分布

* $n$重伯努利实验，事件发生次数$k$的统计规律

* 概率函数

$$P(x) = C_{n}^{x}p^x(1 - p)^{n-x}$$

* 期望

$$E(x) = np$$

* 方差

$$D(x) = np(1 - p)$$

## 3.泊松分布

* 概率函数

$$P(x) = \frac{\lambda^xe^{-\lambda}}{x!}$$
$x$代表事件发生的次数，$\lambda$代表给定时间范围内事件发生的平均次数

* 期望&方差

泊松分布的重要性质是期望和方差都等于$\lambda$

* 性质

$$\lim_{n\rightarrow\infty}C_{n}^{x}p_{n}^{x}(1 - p_n)^{n-x}=\frac{\lambda^xe^{-\lambda}}{x!}$$

在`二项分布`中，若$n$很大,$p$很小时，$np = \lambda$，`二项分布`的概率值与`泊松分布`的概率值近似

---

# 连续随机变量及概率密度

* 对于随机变量$X$的分布函数$F(x)$，存在非负函数$f(x)$，对于任意实数$x$：

$${F(x) = \int ^x_{-\infty}f(t)dt}$$

则$X$为连续型随机变量，其中函数$f(x)$为`概率密度函数`

* 若$f(x)$在$x$点处连续，则$F'(x) = f(x)$，由此可说明`概率密度函数`名称的意义。

* 数学期望

$$E(x) = \int_{-\infty}^{\infty}xf(x)dx$$

## 1.均匀分布

* 概率密度函数：

$$ f(x) =
\begin{cases}
\frac{1}{b-a} , & a \lt x \lt b \\
0,     & x \lt a, x \gt b
\end{cases}
$$

* 分布函数：

$$ F(x) =
\begin{cases}
0, & x \lt a \\
\frac{x-a}{b-a}, & a \le x \lt b \\
1, & x \ge b
\end{cases}
$$

* 数学期望

$$E(x) = \frac{a+b}{2}$$

## 2.指数分布

* 概率密度函数：

$$ f(x) =
\begin{cases}
\frac{1}{\theta}e^{-x/\theta} , & x \gt 0 \\
0,     & x \lt 0
\end{cases}
$$

* 分布函数：

$$ F(x) =
\begin{cases}
1 - e^{-x/\theta}, & x \gt 0 \\
0, & x \lt 0
\end{cases}
$$

* 无记忆性

## 3.正态分布

* 概率密度函数
$$f(x) = \frac{1}{\sqrt{2\pi}\sigma}e^{-(x-u)^2/2\sigma^2}, -\infty \lt x \lt \infty$$

* 分布函数
$$F(x) = \frac{1}{\sqrt{2\pi}\sigma}\int^x_{-\infty}e^{(t-u)^2/2\sigma^2}dt$$

* 均值$u$，方差$\sigma^2$
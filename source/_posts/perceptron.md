title: 感知机
date: 2018-04-23 10:26:43
tags: MachineLearning
mathjax: true
---

```
感知机模型
```
<!-- more -->
# 感知机模型
感知机是二类分类的线性分类模型，输入为样本的特征向量，输出为样本类别，取+1和-1二值，属于判别模型。
* 输入空间到输出空间函数：
$$f(x) = sign(w*x + b)$$
其中$w$为权值，$b$为偏置，$w * x$表示向量内积，$sign$是指符号函数：
$$ sign(x) =
\begin{cases}
+1, & x \ge 0 \\
-1, & x \lt 0
\end{cases}
$$
* 损失函数
$$minL(w, b) = - \sum_{x_i}{y_i}(w * x + b)$$
* 使用`随机梯度下降`法进行对$w$,$b$进行更新
$$w = w + \eta{y_i}{x_i}$$
$$b = b + \eta{y_i}$$
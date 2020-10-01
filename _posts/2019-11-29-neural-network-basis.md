---
layout:     post
title:      Neural Network Basis
subtitle:   DMC报告-2019秋季学期
date:       2019-11-19
author:     Yimeng Ren
header-img: 
catalog: true
tags:
    - Deep Learning
    - Neural Network
---

## Neuron Structure

![典型神经元结构](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9yf788rcj30pa0hdwgb.jpg)

---

## Neuron Structure

\begin{align*}
  z &= \sum_{i=1}^{d} w_{i} x_{i}+b \\
    &= w^{T} x+b \\
  a &= f(z)
\end{align*}

- 其中，`$w=[w_1, w_2, ...,w_d] \in \mathbb{R}^d$` 是d维的权重向量，`$b \in \mathbb{R}$` 是偏置。

- 非线性函数 `$f(\centerdot)$` 称为**激活函数**（Activation Function）

# Common Activation Functions

## Sigmoid

- 两端饱和
- Logistic Function

\begin{align*}
  \sigma(x)=\frac{1}{1+\exp (-x)}
\end{align*}

- Tanh Function

\begin{align*}
  \tanh (x)=\frac{\exp (x)-\exp (-x)}{\exp (x)+\exp (-x)}
\end{align*}

- Tanh可以看作是放大并平移的Logistic，值域为 (-1,1)。

---

## Sigmoid

![Logistic函数和Tanh函数](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9yg4n8upj30ku0ixab5.jpg)

---

## Hard Sigmoid

- `$exp()$` 的求导计算量较大
- 对Logistic函数在0附近做一阶泰勒展开

\begin{align*} 
g_{l}(x) & \approx \sigma(0)+x \times \sigma^{\prime}(0) \\ &=0.25 x+0.5 
\end{align*}

- 此时可以用分段函数 `$hard-logistic(x)$` 来近似：

\begin{align*} 
hard-logistic(x) &=\left\{\begin{array}{ll}{1} & {g_{l}(x) \geq 1} \\ {g_{l}} & {0<g_{l}(x)<1}\end{array}\right.\\ &=\max \left(\min \left(g_{l}(x), 1\right), 0\right) \\ &=\max (\min (0.25 x+0.5,1), 0) 
\end{align*}

## Hard Sigmoid

![Hard Sigmoid型激活函数](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9ygnj1brj30q00d4wg2.jpg)

## Rectified Linear Unit (ReLU) ^[@nair2010rectified]

- 比Sigmoid函数具有更好的稀疏性，大约50%的神经元处于激活状态
- `$x>0$`时导数为1，一定程度缓解梯度消失问题

\begin{align*} 
\operatorname{ReLU}(x) &=\left\{\begin{array}{ll}{x} & {x \geq 0} \\ {0} & {x<0}\end{array}\right.\\ &=\max (0, x) \\
\Delta_{x} \operatorname{ReLU}(x) &=1(x>0)
\end{align*}

- **Dying ReLU Problem**: 如果某种情况下（e.g. 大的梯度更新导致ReLU的输入小于0），部分输入`$W$`经过ReLU函数能得到一个0（ReLU is close），那么反向传播时，`$W$`一般都不能通过反向传播得到更新。

## ReLU Variant^[@Maas2013RectifierNI; @clevert2015fast; @dugas2000incorporating]

![ReLU、Leaky ReLU、ELU以及Softplus函数](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9yhv0brtj30oa0kn75v.jpg)

# Network Architectures

## Universal Approximation Theorem ^[@hornik1989multilayer]

> A feed-forward network with a single hidden layer containing a finite number of neurons can approximate continuous functions on compact subsets of $R$, under mild assumptions on the activation function.

- Why deep?

  - 不同隐藏层中，不同复杂度的特征学习；
  - 实际数据表现^[@yu2012conversational]：相同参数数量下，深度网络表现更好；这也就意味着，达到相同的效果，深度网络的参数会更少；

## Network Architecture

- **前馈网络**

  - 非线性函数的多次复合，整个网络的信息只往一个方向传播，不反向传播
  - 全连接前馈网络、卷积神经网络（CNN）

- 记忆网络

  - 不但可以接收其它神经元的信息，也可以接收自己的历史信息
  - **循环神经网络（RNN）**、Hopfield网络、玻尔兹曼机（Boltzmann Machine）、受限玻尔兹曼机（RBM）

- 图网络

  - 处理图结构的数据（知识图谱、社交网络、分子网络）
  - 图中每个节点都由一个或一组神经元构成。节点之间的连接可以是有向的，也可以是无向的。每个节点可以收到来自相邻节点或自身的信息。
  - 图卷积神经网络（GCN）、Graph Attention Networks (GAT)、Message Passing Neural Network (MPNN)

## Network Architectures

![三种不同的网络结构示例](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9yi924a8j31660doael.jpg)

## Feedforward Neural Network (FNN)

\begin{align*} 
  \boldsymbol{a}^{(l)} &=f_{l}\left(\boldsymbol{z}^{(l)}\right) = f_{l} \left(W^{(l)} \cdot \boldsymbol{a}^{(l-1)}+\boldsymbol{b}^{(l)}\right) \\
  \boldsymbol{x} &=\boldsymbol{a}^{(0)} \rightarrow \boldsymbol{z}^{(1)} \rightarrow \boldsymbol{a}^{(1)} \rightarrow \boldsymbol{z}^{(2)} \rightarrow \cdots \rightarrow \boldsymbol{a}^{(L-1)} \rightarrow \boldsymbol{z}^{(L)} \rightarrow \boldsymbol{a}^{(L)}=\phi(\boldsymbol{x} ; W, \boldsymbol{b})
\end{align*}

![多层前馈神经网络](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9yij5ckwj30xf0h3tej.jpg)

# Back Propagation

## Review and Notation

- 梯度下降（Gradient Descent）

  - BGD、SGD、MBGD
  - SGD Variant: SAG、SVRG、OCO, etc.

- 符号说明
  - $L$：表示神经网络的层数；
  - $m^{(l)}$：表示第$l$层神经元的个数；
  - $f_l(·)$：表示第$l$层神经元的激活函数；
  - $W^{(l)} \in \mathbb{R}^{m^{(l)} \times m^{l-1}}$：表示第$l−1$层到第$l$层的权重矩阵；
  - $\boldsymbol{b}^{(l)} \in \mathbb{R}^{m^{l}}$：表示第$l−1$层到第$l$层的偏移项；
  - $\boldsymbol{z}^{(l)} \in \mathbb{R}^{m^{l}}$：表示第$l$层神经元的净输入；
  - $\boldsymbol{a}^{(l)} \in \mathbb{R}^{m^{l}}$：表示第$l$层神经元的输出；

## Loss Function

- 如果使用交叉熵损失函数，对于样本 $(x,y)$，其损失函数为：

\begin{align*}
\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}}) &=-\boldsymbol{y}^{\mathrm{T}} \log \hat{\boldsymbol{y}} \\
\mathcal{R}(W, \boldsymbol{b}) &=\frac{1}{N} \sum_{n=1}^{N} \mathcal{L}\left(\boldsymbol{y}^{(n)}, \hat{\boldsymbol{y}}^{(n)}\right)+\frac{1}{2} \lambda\|W\|_{F}^{2} \\
\|W\|_{F}^{2} &=\sum_{l=1}^{L} \sum_{i=1}^{m^{(l)}} \sum_{j=1}^{m^{(l-1)}}\left(w_{i j}^{(l)}\right)^{2}
\end{align*}

## Gradiant Descend

\begin{align*} 
W^{(l)} & \leftarrow W^{(l)}-\alpha \frac{\partial \mathcal{R}(W, \boldsymbol{b})}{\partial W^{(l)}} \\ &=W^{(l)}-\alpha\left(\frac{1}{N} \sum_{n=1}^{N}\left(\frac{\partial \mathcal{L}\left(\boldsymbol{y}^{(n)}, \hat{\boldsymbol{y}}^{(n)}\right)}{\partial W^{(l)}}\right)+\lambda W^{(l)}\right) \\
\boldsymbol{b}^{(l)} & \leftarrow \boldsymbol{b}^{(l)}-\alpha \frac{\partial \mathcal{R}(W, \boldsymbol{b})}{\partial \boldsymbol{b}^{(l)}} \\ &=\boldsymbol{b}^{(l)}-\alpha\left(\frac{1}{N} \sum_{n=1}^{N} \frac{\partial \mathcal{L}\left(\boldsymbol{y}^{(n)}, \hat{\boldsymbol{y}}^{(n)}\right)}{\partial \boldsymbol{b}^{(l)}}\right)
\end{align*}

## Gradiant Descend

- 对第$l$层中的参数$W^{(l)}$ 和$b^{(l)}$ 计算偏导数：

\begin{align*}
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial w_{i j}^{(l)}}=\frac{\partial \boldsymbol{z}^{(l)}}{\partial w_{i j}^{(l)}} \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} \\
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{b}^{(l)}}=\frac{\partial \boldsymbol{z}^{(l)}}{\partial \boldsymbol{b}^{(l)}} \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}}
\end{align*}

## Partial Derivative 1

- 计算偏导数`$\frac{\partial z^{(l)}}{\partial w_{ij}^{(l)}}$，因为$z^{(l)}=W^{(l)}a^{(l-1)}+b^{(l)}$`，偏导数

\begin{align*} 
\frac{\partial \boldsymbol{z}^{(l)}}{\partial w_{i j}^{(l)}} &=\left[\frac{\partial z_{1}^{(l)}}{\partial w_{i j}^{(l)}}, \cdots, \frac{\partial z_{i}^{(l)}}{\partial w_{i j}^{(l)}}, \cdots, \frac{\partial z_{m^{(l)}}^{(l)}}{\partial w_{i j}^{(l)}}\right] \\ 
&=\left[0, \cdots, \quad \frac{\partial\left(\boldsymbol{w}_{i:}^{(l)} \boldsymbol{a}^{(l-1)}+b_{i}^{(l)}\right)}{\partial w_{i j}^{(l)}}, \cdots, 0\right] \\ 
&=\left[0, \cdots, a_{j}^{(l-1)}, \cdots, 0\right] \\
&\triangleq \mathbb{I}_{i}\left(a_{j}^{(l-1)}\right) \quad \in \mathbb{R}^{m^{(l)}}
\end{align*}

## Partial Derivative 2

- 计算偏导数`$\frac{\partial z^{(l)}}{\partial b^{(l)}}$，因为$z^{(l)}=W^{(l)}a^{(l-1)}+b^{(l)}$`，偏导数

\begin{align*}
\frac{\partial \boldsymbol{z}^{(l)}}{\partial \boldsymbol{b}^{(l)}}=\boldsymbol{I}_{m^{(l)}} \quad \in \mathbb{R}^{m^{(l)} \times m^{(l)}}
\end{align*}

- 求导结果为`$m^{(l)}*m^{(l)}$`的单位矩阵。

## Partial Derivative 3

- 计算偏导数$\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} \triangleq \delta^{(l)}$

- 这个值表示第$l$层神经元对最终损失的影响（贡献程度），也反映了最终损失对第$l$层神经元的敏感程度，因此一般称为第$l$层神经元的**误差项**，用$\delta^{(l)}$来表示。

- 根据$z^{(l+1)}=W^{(l+1)}a^{(l)}+b^{(l+1)}$，有$\frac{\partial z^{(l+1)}}{\partial a^{(l)}}=\left(W^{(l+1)}\right)^{\mathrm{T}}$

- 根据$\boldsymbol{a}^{(l)}=f_{l}\left(\boldsymbol{z}^{(l)}\right)$，其中$f_{l}(\centerdot)$按照位来计算，因此有

\begin{align*} 
\frac{\partial \boldsymbol{a}^{(l)}}{\partial \boldsymbol{z}^{(l)}} &=\frac{\partial f_{l}\left(\boldsymbol{z}^{(l)}\right)}{\partial \boldsymbol{z}^{(l)}} \\ &=\operatorname{diag}\left(f_{l}^{\prime}\left(\boldsymbol{z}^{(l)}\right)\right) 
\end{align*}

## Partial Derivative 3

- 根据链式法则，第`$l$`层的误差项为

\begin{align*} 
\delta^{(l)} & \triangleq \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} \\ &=\frac{\partial \boldsymbol{a}^{(l)}}{\partial \boldsymbol{z}^{(l)}} \cdot \frac{\partial \boldsymbol{z}^{(l+1)}}{\partial \boldsymbol{a}^{(l)}} \cdot \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l+1)}} \\
&=\operatorname{diag}\left(f_{l}^{\prime}\left(z^{(l)}\right)\right) \cdot\left(W^{(l+1)}\right)^{\mathrm{T}} \cdot \delta^{(l+1)} \\ 
&=f_{l}^{\prime}\left(z^{(l)}\right) \odot\left(\left(W^{(l+1)}\right)^{\mathrm{T}} \delta^{(l+1)}\right)
\end{align*}

- 其中`$\odot$`是向量的点积运算符，表示每个元素相乘；

- 理解反向传播算法：第$l$层的一个神经元的误差项（或敏感性）是所有与该神经元相连的第$l+1$层的神经元的误差项的权重和。然后再乘上该神经元激活函数的梯度。 

## Partial Derivative

- 计算出上面三个偏导数之后，损失函数对一个权重`$w_{ij}^{(l)}$`的梯度就可以写成

\begin{align*} 
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial w_{i j}^{(l)}} &=\mathbb{I}_{i}\left(a_{j}^{(l-1)}\right) \delta^{(l)} \\ &=\left[0, \cdots, a_{j}^{(l-1)}, \cdots, 0\right]\left[\delta_{1}^{(l)}, \cdots, \delta_{i}^{(l)}, \cdots, \delta_{m^{(l)}}^{(l)}\right]^{\mathrm{T}} \\ &=\delta_{i}^{(l)} a_{j}^{(l-1)} 
\end{align*}

- 其中$\delta_{i}^{(l)} a_{j}^{(l-1)}$相当于向量$\delta_{i}^{(l)}$和向量$a_{j}^{(l-1)}$的**外积**的第{i,j}个元素，因此上式进一步写成

\begin{align*}
\left[\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial W^{(l)}}\right]_{i j}=\left[\delta^{(l)}\left(\boldsymbol{a}^{(l-1)}\right)^{\mathrm{T}}\right]_{i j}
\end{align*}

## Gradient

- $\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})$关于第$l$层权重$W^{(l)}$的梯度为：

\begin{align*}
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial W^{(l)}}=\delta^{(l)}\left(\boldsymbol{a}^{(l-1)}\right)^{\mathrm{T}}
\end{align*}

- $\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})$关于第$l$层偏移项$b^{(l)}$的梯度为：

\begin{align*}
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{b}^{(l)}}=\delta^{(l)}
\end{align*}

## Back Propagation

![反向传播算法](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9yjttzcij30o20mfn1r.jpg)

# Regularization

## Regularization

- 训练数据集上的经验风险最小化和样本真实分布上的**期望风险**并不一致

- 神经网络的拟合能力非常强，其在训练数据上的错误率往往都可以降到非常低，甚至可以到0，从而导致过拟合

- 正则化：限制模型复杂度

  - 传统机器学习：$l1,l2$正则化
  - 深度神经网络：由于过度参数化的问题（over-parameterization），一般使用数据增强（Data Augmentation）、权重衰减（Weight Decay）、提前停止（Early Stop）、**丢弃法**（Dropout）等
  
## Weight Decay^[@hanson1988comparing]

- 在每次参数更新时，引入一个衰减系数

\begin{align*}
\theta_{t} \leftarrow(1-w) \theta_{t-1}-\alpha \mathbf{g}_{t}
\end{align*}

- 在标准的随机梯度下降中，权重衰减正则化和$l2$正则化的效果相同。因此，权重衰减在一些深度学习框架中通过$l2$正则化来实现。但是，在较为复杂的优化方法（比如Adam）中，权重衰减和$l2$正则化并不等价^[@DBLP:journals/corr/abs-1711-05101]。

## Early Stop

- 在使用梯度下降法进行优化时，我们可以使用一个和训练集独立的样本集合，称为验证集（Validation Set），并用验证集上的错误来代替期望错误。
- 当验证集上的错误率不再下降，就停止迭代^[@prechelt1998early]。

## Dropout

- 当训练一个深度神经网络时，我们可以随机丢弃一部分神经元（同时丢弃 其对应的连接边）来避免过拟合^[@srivastava2014dropout; @wan2013regularization]。
- 对于一个神经层$y = f(Wx + b)$，我们可以引入一个丢弃函数$d(·)$，使得$y = f(Wd(x)+ b)$。丢弃函数$d(·)$的定义为 

\begin{align*}
\operatorname{d}(\boldsymbol{x})=\left\{\begin{array}{ll}{\boldsymbol{m} \odot \boldsymbol{x}} & {\text { Training Set }} \\ {p \boldsymbol{x}} & {\text { Test Set }}\end{array}\right.
\end{align*}

- 其中`$m \in \{0,1\}^d$`是丢弃掩码（Dropout Mask），通过以概率为$p$的伯努利分布随机生成。

## Dropout

- 对于隐藏层的神经元，其丢弃率p = 0.5时效果最好
- 对于输入层的神经元，其丢弃率通常设为更接近1的数，使得输入变化不会太大
- 丢弃法一般是针对神经元进行随机丢弃，也可以扩展到对神经元之间的连接进行随机丢弃，或每一层进行随机丢弃

![丢弃法示例](https://tva1.sinaimg.cn/large/007S8ZIlly1gj9ykfsw5sj30zc0iwq9d.jpg)

## Two Explanations for Dropout

- 集成学习角度

  - 每做一次丢弃，相当于从原始的网络中采样得到一个子网络。如果一个神经网络有`$n$`个神经元，那么总共可以采样出`$2^n$`个子网络。 

- 贝叶斯学习角度^[@gal2016dropout]
  - 贝叶斯学习是假设参数`$\theta$`为随机向量，并且先验分布为`$q(\theta)$`，贝叶斯方法的预测为 

\begin{align*} 
\mathbb{E}_{q(\theta)}[y] &=\int_{q} f(\boldsymbol{x} ; \theta) q(\theta) d \theta \approx \frac{1}{M} \sum_{m=1}^{M} f\left(\boldsymbol{x}, \theta_{m}\right) 
\end{align*}

- 其中$f(x,\theta_m)$为第$m$次应用丢弃方法后的网络，其参数$\theta_m$为对全部参数$\theta$的一次采样。

## Data Augmentation

- 在数据量有限的情况下，可以通过数据增强来增加数据量，提高模型鲁棒性，避免过拟合。
- 主要运用在图像处理中，对图像进行转变，引入噪声等方法来增加数据的多样性。
  - 旋转（rotation）、翻转（flip）、缩放（scale）、裁剪（crop）、平移（translation）、加噪声（noise）^[https://medium.com/nanonets/nanonets-how-to-use-deep-learning-when-you-have-limited-data-f68c0b512cab]

## References {.allowframebreaks}

\footnotesize

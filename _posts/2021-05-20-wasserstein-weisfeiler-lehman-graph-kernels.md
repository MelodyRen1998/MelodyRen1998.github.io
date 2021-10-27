---
title: '[论文阅读] Wasserstein Weisfeiler-Lehman Graph Kernels'
author: yimeng
date: '2021-05-20'
permalink: /posts/2021/05/wasserstein-weisfeiler-lehman-graph-kernels/
categories:
  - notes
tags:
  - graph-neural-network
  - graph kernel
  - wasserstein distance
---

## Graph Similarity Learning

Ma, G., Ahmed, N. K., Willke, T. L., & Philip, S. Y. (2021). Deep graph similarity learning: A survey. *Data Mining and Knowledge Discovery*, 1-38.

- encoder: 把输入的graph映射到嵌入向量的空间
- decoder: 使得目标空间中的距离可以近似输入的graph结构之间的"距离"

### 传统方法

- 图同构问题（graph isomorphism）：NP

- 子图同构问题（subgraph isomorphism）：NP-complete
- 最大共同子图（max common subgraph）：实际上是子图同构问题的变体

### deep graph similarity方法

1. graph embedding based
2. graph neural network based
3. deep graph kernel based

#### graph embedding based

graph embedding 是在 similarity score 之前学习的，因此并非端到端的学习方法。

**SNA: **

**Graph classification with 2d convolutional neural network.**（Tixier, A. J. P., Nikolentzos, G., Meladianos, P., & Vazirgiannis, M. (2019, September). Graph classification with 2d convolutional neural networks. In *International Conference on Artificial Neural Networks* (pp. 578-593). Springer, Cham.）

**Inductive and unsupervised representation learning on graph structured objects.**（Wang, L., Zong, B., Ma, Q., Cheng, W., Ni, J., Yu, W., ... & Fu, Y. (2020). Inductive and Unsupervised Representation Learning on Graph Structured Objects. In *ICLR*.）

#### GNN based

包括之前讲的GMN模型，以graph embedding + similarity score为结构，最终得到score回传更新模型参数。

#### deep graph kernel based

目标：学习一个kernel function，捕捉两个图之间的相似性。

**SNA:**

**Graph neural tangent kernel: Fusing graph neural networks with graph kernels.**（Du, S. S., Hou, K., Póczos, B., Salakhutdinov, R., Wang, R., & Xu, K. (2019). Graph neural tangent kernel: Fusing graph neural networks with graph kernels. *arXiv preprint arXiv:1905.13192*.）

**Deep graph kernels.**（Yanardag, P., & Vishwanathan, S. V. N. (2015, August). Deep graph kernels. In *Proceedings of the 21th ACM SIGKDD international conference on knowledge discovery and data mining* (pp. 1365-1374).）

## Wasserstein Distance

作用：衡量概率分布之间的差异。

- 能够很自然地度量离散分布和连续分布之间的距离；
- 不仅给出了距离的度量，而且给出如何把一个分布变换为另一分布的方法；
- 能够连续地把一个分布变换为另一个分布，在此同时，能够保持分布自身的几何形态特征；

### 定义

Wasserstein距离的起源是optimal transport problem，把概率分布想象成一堆石子，这个问题关心如何移动一堆石子，通过最小的累积移动距离把它堆成另外一个目标形状。

首先，要能完成这个操作，先要确保本来的这一堆石子的总质量要和目标石子堆总质量一样——概率分布的归一化条件。

其次，我们暂时假设石子都是很小的，无限可分的。

假设地面上 $\mathcal{X}=\mathbb{R}^{2}$ 堆了一些石子，石子的分布我们用 $\mu: \mathcal{X} \rightarrow \mathbb{R}$ 来表示，采取这样的表示方法对于地面上的任意一块面积 $A \subseteq \mathcal{X}$,$ \mu(A)$ 表示这块面积上放置了质量为多少的石子。同样的我们可以定义目标石子堆的分布 $\nu$。定义一个输运方案 $T: \mathcal{X} \rightarrow \mathcal{X}$ 把现有的石子堆变成目标石子堆。$T(A)=B$ 表示把原来放在A处的石子都运到B处放好，类似地可以定义反函数 $T^{-1}(B)=A$。该输运方案成立需要满足 $\nu(B)=\mu\left(T^{-1}(B)\right), \forall B \subseteq \mathcal{X}$，即任意位置的石子通过输运过后都刚好满足分布 $\mu$ 的要求。这也可以写为 $T \# \mu=\nu$。

两堆石子之间的距离可以被定义成把一堆石子挪动成另外一堆所需要的最小输运成本：

$$
W_{p}(\mu, \nu)=\left(\inf _{\gamma \in \Gamma(\mu, \nu)} \int_{\mathcal{X} \times \mathcal{X}}\|x-y\|^{p} d \gamma(x, y)\right)^{1 / p}
$$

其中 $\gamma$ 是一个联合概率分布，称coupling，它要求其边缘分布刚好是 $\mu$ 和 $\nu$，即 $\gamma(A \times \mathcal{X})=\mu(A), \quad \gamma(\mathcal{X} \times B)=\nu(B)$

![分布$\mu$上的某个位置的质量被拆开（红竖线），然后再按照权重被分配给目标分布（红箭头）](https://tva1.sinaimg.cn/large/008i3skNly1gqmgt8s51pj30ki0jqaf2.jpg)

## Graph Kernel 与 Wassertein distance

Togninalli, M., Ghisu, E., Llinares-López, F., Rieck, B., & Borgwardt, K. (2019). Wasserstein weisfeiler-lehman graph kernels. *arXiv preprint arXiv:1906.01277*.

**info**: [Advances in Neural Information Processing Systems 32 (NeurIPS 2019)](https://papers.nips.cc/paper/2019)

DEPARTMENT OF BIOSYSTEMS SCIENCE AND ENGINEERING, ETH ZURICH, SWITZERLAND

这篇论文里，作者提出一个更好的 graph kernel 来度量两个图之间的相似度，基于两个图 node feature representations，提出了两个图之间的 graph Wasserstein distance。受 Weisfeiler-Lehman 启发，并且融入了 Wasserstein distance（两个 graph 的 node feature vector distribution 之间），提出了新的图嵌入的学习方法。

**Notation:** 一个图表示为 $G=(V,E)$。其中 $V$ 为点集，$E$ 为边集。$n_{G}=|V|, \quad m_{G}=|E|$。

目前大部分的graph kernels定义方法都是使用子结构集合的 naive aggregation (e.g. sum or average)，这样会丢掉子结构的分布等重要信息，因此借鉴Wassertein距离的思路可以避免这种过分简化带来的信息损失。

回忆$L^p-Wassertein \ Distance$的定义：
$$
W_{p}(\sigma, \mu):=\left(\inf _{\gamma \in \Gamma(\sigma, \mu)} \int_{M \times M} d(x, y)^{p} \mathrm{~d} \gamma(x, y)\right)^{\frac{1}{p}}
$$
由于node embeddings是有限集，而非连续的概率分布，因此上式中积分的部分可以表示为求和，进而使用矩阵乘积的方式给出两个向量集合$X \in \mathbb{R}^{n \times m}$ 和 $X^{\prime} \in \mathbb{R}^{n^{\prime} \times m}$的距离：
$$
W_{1}\left(X, X^{\prime}\right):=\min _{P \in \Gamma\left(X, X^{\prime}\right)}\langle P, M\rangle
$$
其中，$M$是距离矩阵，$P$是传输矩阵（可以视作联合概率），$\langle\cdot, \cdot\rangle$表示Frobenius点积运算。假设 需要被传输的石头质量之和为1，并且在$X$和$X'$上均匀分布，因此$P$的行和与列和分别为$1/n$和$1/n'$。

### 算法

![](https://tva1.sinaimg.cn/large/008i3skNly1gqmmtg9omxj30x508owgp.jpg)

#### 第一步：生成node embeddings

1. labelled no-attributed graph

第$h+1$轮迭代得到的label vector表达式为：$\ell^{h+1}(v)=\operatorname{hash}\left(\ell^{h}(v), \mathcal{N}^{h}(v)\right)$，可以参考[这篇博客](https://yimeng.netlify.app/2021/05/06/neural-subgraph-matching-math-test/)中的WL TEST部分。

2. graph with continuous attributes $a(v)$

第$h+1$轮迭代得到的attribute vector表示为：$a^{h+1}(v)=\frac{1}{2}\left(a^{h}(v)+\frac{1}{\operatorname{deg}(v)} \sum_{u \in \mathcal{N}(v)} w((v, u)) \cdot a^{h}(u)\right)$

通过上面两种情况迭代$H$轮得到结果后，构造$WL \ features$: $X_{G}^{h}=\left[x^{h}\left(v_{1}\right), \ldots, x^{h}\left(v_{n_{G}}\right)\right]^{T}$
$$
\begin{aligned}
f^{H}: G & \rightarrow \mathbb{R}^{n_{G} \times(m(H+1))} \\
G & \mapsto \text { concatenate }\left(X_{G}^{0}, \ldots, X_{G}^{H}\right)
\end{aligned}
$$

#### 第二步：构造Wasserstein distance

首先构造groud distance (Wassertein Distance中的D矩阵)。如果是第一种情况，用hamming distance；如果是第二种情况则用欧式距离。
$$
d_{\mathrm{Ham}}\left(v, v^{\prime}\right)=\frac{1}{H+1} \sum_{i=1}^{H+1} \rho\left(v_{i}, v_{i}^{\prime}\right), \quad \rho(x, y)=\left\{\begin{array}{l}
1, x \neq y \\
0, x=y
\end{array}\right.
$$

$$
d_{E}\left(v, v^{\prime}\right)=\left\|v-v^{\prime}\right\|_{2}
$$

#### 第三步：计算WWL kernel

$$
K_{\mathrm{WWL}}=e^{-\lambda D_{W}^{f \mathrm{WL}}}
$$

### 性质

用于分类特征的 WWL kernel 对于所有的 $\lambda>0$ 都是正定的。

### 实验

![](https://tva1.sinaimg.cn/large/008i3skNly1gqo6jvfldqj310l0cwn0q.jpg)

## TODO

Kolouri, S., Naderializadeh, N., Rohde, G. K., & Hoffmann, H. (2020). Wasserstein Embedding for Graph Learning. *arXiv preprint arXiv:2006.09430*.


## 参考资料

【数学】Wasserstein Distance - 张楚珩的文章 - 知乎 https://zhuanlan.zhihu.com/p/58506295

https://blog.csdn.net/weixin_40239306/article/details/108930560

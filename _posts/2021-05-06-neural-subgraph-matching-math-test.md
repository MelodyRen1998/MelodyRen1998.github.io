---
title: '[论文阅读] Neural Subgraph Matching'
author: Yimeng
date: '2021-05-06'
permalink: /posts/2021/05/neural-subgraph-matching-math-test/
categories:
  - notes
tags:
  - graph-neural-network
---

![](https://tva1.sinaimg.cn/large/008i3skNgy1gq6s1j63gzj31eu0gawkx.jpg)

作者：[Rex Ying](http://cs.stanford.edu/people/rexy/), [Joe Lou](http://stanford.edu/~zlou/), [Jiaxuan You](http://cs.stanford.edu/people/jiaxuan/), Chengtao Wen (Arquimedes Canedo from Siemens, [Jure Leskovec](http://cs.stanford.edu/people/jure/)

![](https://cs.stanford.edu/people/jure/images/jure-6.jpg){width=40%} 

## foreword: Weisfeiler-Lehman (WL) test[^1]

用于判断图是否同构（graph isomorphism）。

![](https://tva1.sinaimg.cn/large/008i3skNly1gq8j3plz36j30ue0r5jwu.jpg)

- 网络中每个节点有一个label，如图中的彩色的1，2，3，4，5
- 标签扩展：做一阶(1-hop)广度优先搜索，即只遍历自己的邻居。比如在图（a）网络G中原(5)号节点，变成(5,234)
- 标签压缩：仅仅只是把扩展标签映射成一个新标签，如 5,234 => 13
- 压缩标签替换扩展标签
- 数标签：比如在G网络中，含有1号标签2个，那么第一个数字就是2。这些标签的个数作为整个网络的新特征

## term

target graph: 大图

query graph: 小图

给定一个query，NeuroMatch的目标是在target graph中找到一些节点，这些节点的K-hop邻居包含了query。

将包含这个query的大图中所有节点的N-hop邻域识别为子图（subgraph）【可以理解为一个节点识别问题】。利用得到的子图，NeuroMatch使用一个GNN来学习每个节点的embedding，进而根据query和子图的embedding相似与否来刻画子图的结构与性质。

## motivation

子图（subgraph）的结构同质性匹配是一个NP-complete问题，但是又有非常广泛的应用：

- social science: 子图分析在分析社会网络中的网络效应方面发挥了重要作用；
- Information retreival systems: 利用知识图中的子图结构进行语义总结、类比推理和关系预测；
- Chemistry: 子图匹配是一种确定化合物间相似性的稳健而准确的方法；
- Biology: 子图匹配在蛋白质相互作用网络的分析中具有中心重要性，识别和预测功能基序是理解诸如潜在疾病、衰老和医学等生物学机制的主要工具。

## problems

1. 给定一个target graph和一个query graph，判断query graph是否和target graph的一个子图（子图即按照上述的搜索方式来判断）是同构的。
2. 给定一个节点u的邻居子图，和中心化（by **anchor node** q）之后的query graph（ $G_q$ ），以二分类任务预测这个$G_q$是否是前者$G_u$的一个子图（q和u是一一匹配的）

## NeuroMatch

### Embedding Stage

对target graph中的节点u，使用GNN得到$z_u$。这里作者使用GIN[^2]模型的变体，GIN的效果和Weisfeiler-Lehman (WL) test一样有效，但是变体GIN比WL test更有效。

### Query Stage

目标：确定$G_Q$是不是$G_T$的子图

问题转化：确定$G_q$是不是$G_u$的子图

方法：构造一个“子图预测函数”$f\left(z_{q}, z_{u}\right)$，进行二分类预测

![](https://tva1.sinaimg.cn/large/008i3skNly1gq8exf7abdj313r09s771.jpg)

### practical consideration

k-hop邻居选取时，k的设定至少等于query graph的直径。

## predict function

使用order embedding的方式，保证在通过子图结构构造embedding之后，得到的向量可以体现子图关系性质。具体地，考虑以下4个性质：

- 传递性
- 对称性
- 交集性质
- 非平凡交集（每两个图的交集至少含有一个trivial graph）

训练embedding的目标函数：最大边际损失
$$
\begin{array}{c}
\mathcal{L}\left(z_{q}, z_{u}\right)=\sum_{\left(z_{q}, z_{u}\right) \in P} E\left(z_{q}, z_{u}\right)+\sum_{\left(z_{q}, z_{u}\right) \in N} \max \left\{0, \alpha-E\left(z_{q}, z_{u}\right)\right\}, \text { where } \\
E\left(z_{q}, z_{u}\right)=\left\|\max \left\{0, z_{q}-z_{u}\right\}\right\|_{2}^{2}
\end{array}
$$
进而构造子图预测函数
$$
f\left(z_{q}, z_{u}\right)=\left\{\begin{array}{ll}
1 & \text { iff } E\left(z_{q}, z_{u}\right)<t \\
0 & \text { otherwise }
\end{array}\right.
$$

## matching nodes

$\mathcal{N}^{(k)}(u)$表示节点u的kth-hop邻居。如果$q \in G_q$和节点$u \in G_u$ 是匹配的，那么所有的节点$i \in N^{(k)}(q)$，存在节点$j \in \mathcal{N}^{(l)}(u), l \leq k$，使得节点i和节点j是匹配的。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gq6sqvs141j30to0n6wj6.jpg)

## training procedure

为了使得整个模型inductive，即可以在query graph未参与训练的情况下仍然有较高的预测精度，在训练时采用随机生成的query graph。

NeuroMatch是有监督的训练过程：构造positive pair和negative pair

- positive pair: 抽样$G_{u} \in G_{T}$，$G_{q} \in G_{u}$
- negative pair: 随便选取或直接打乱positive pair

## experiment

### dataset

合成数据集：Renyi random graphs,  extended Barabasi graphs

真实数据集：（化学）COX2，（生物）Enzymes, DD, PPI networks，（图像处理）MSRC_21，（点云？）FIRSTMMDB，（知识图谱）WORDNET18

### baseline

Graph Matching Neural Networks

> Kun Xu, Liwei Wang, Mo Yu, Yansong Feng, Yan Song, Zhiguo Wang, and Dong Yu. Cross-lingual knowledge graph alignment via graph matching neural network. 2019.

 RDGCN

> Yuting Wu, Xiao Liu, Yansong Feng, Zheng Wang, Rui Yan, and Dongyan Zhao. Relation-aware entity alignment for heterogeneous knowledge graphs. 2019.

## Reference

[^1]: Shervashidze, N., Schweitzer, P., Van Leeuwen, E. J., Mehlhorn, K., & Borgwardt, K. M. (2011). Weisfeiler-lehman graph kernels. Journal of Machine Learning Research, 12(9).

[^2]: Xu, K., Hu, W., Leskovec, J., & Jegelka, S. (2018). How powerful are graph neural networks?. arXiv preprint arXiv:1810.00826.




---
title: '[论文阅读] Graph Attention Network'
author: Yimeng
date: '2021-03-25'
slug: graph-attention-network
permalink: /posts/2021/03/graph-attention-network/
categories:
  - notes
tags:
  - graph-neural-network
---

## Inductive Learning

图数据中的每一个节点可以通过边的关系利用其他节点的信息，这样就产生了一个问题，如果训练集上的节点通过边关联到了预测集或者验证集的节点，那么在训练的时候能否用它们的信息呢? 如果训练时用到了测试集或验证集样本的信息(或者说，测试集和验证集在训练的时候是可见的), 我们把这种学习方式叫做transductive learning, 反之，称为inductive learning. GraphSAGE是一个inductive框架，在具体实现中，训练时它仅仅保留训练样本到训练样本的边。

GraphSAGE采用了邻居节点抽样的机制，克服了GCN训练时内存和显存上的限制，使得图模型可以应用到大规模的图结构数据中。然而，每个节点这么多邻居，采样能否考虑到邻居的相对重要性呢，或者我们在聚合计算中能否考虑到邻居的相对重要性? （GAT）

## 相关研究递进关系

- Semi-Supervised Classification with Graph Convolutional Networks，ICLR 2017，图卷积网络 https://arxiv.org/abs/1609.02907

- Graph Attention Networks，ICLR 2018.  图注意力网络，就是此篇文章所解析的论文 https://arxiv.org/abs/1710.10903

- Relational Graph Attention Networks ，ICLR2019  关联性图注意力网络，整合了GCN+Attention+Relational

GCN做到了将卷积的思想运用到了GNN上，在网络节点划分任务上取得了很好的效果，但是对节点的特征表示却局限于图的结构，因此**对于一个新的图结构我们总是要去训练一个新的模型**。同时GCN是基于图的拉普拉斯矩阵来做卷积操作，其中涉及了大量的矩阵运算。

GAT: 一种将attention机制运用到图结构数据节点分类任务的方法。它结合了attention高并行性的特点，并且对于不同度的节点的不同邻居赋予各自的权值，GAT还能泛化与之前没有训练过的图，直接适用于归纳学习问题。

## Attention

注意力机制是在NLP领域应用比较广泛，推荐系统中的Attention机制理解：通常对不同的item感兴趣程度、注意力分布不同，考虑对不同的item施加不同的权重，即求当前query关于不同key下的注意力分布及当前query的注意力分数。某些特征就会主导某一次的预测，就好像模型对某些特征更加专注。

标准的注意力矩阵由 query (Q) 和 key (K) 矩阵相乘，再通过 softmax 计算得到两两之间的相似得分，进而得到注意力矩阵 A。 
$$
\mathrm{A}(Q, K, V)=\operatorname{softmax}\left(\frac{Q K^{T}}{\sqrt{d}}\right) V
$$

## GAT模型结构

一个单层的graph attentional layer：
$$
\mathbf{h}=\left\{\vec{h}_{1}, \vec{h}_{2}, \ldots, \vec{h}_{N}\right\}, \vec{h}_{i} \in \mathbb{R}^{F}
$$
转换后的输出：
$$
\mathbf{h}^{\prime}=\left\{\vec{h}_{1}^{\prime}, \vec{h}_{2}^{\prime}, \ldots, \vec{h}_{N}^{\prime}\right\}, \vec{h}_{i}^{\prime} \in \mathbb{R}^{F^{\prime}}
$$
首先，要对输入节点做一个线性变换，使节点带有更高表达能力的高级特征。这一步使用一个共享的权值矩阵W并行的应用于每一个节点。之后使用共享的自注意力机制来计算attention系数：
$$
e_{i j}=a\left(\mathbf{W} \vec{h}_{i}, \mathbf{W} \vec{h}_{j}\right)
$$
e_ij 定义了j的特征对i的重要程度，这里我们只计算i的一阶邻居节点j∈Ni。之后softmax归一化：
$$
\alpha_{i j}=\operatorname{softmax}_{j}\left(e_{i j}\right)=\frac{\exp \left(e_{i j}\right)}{\sum_{k \in \mathcal{N}_{i}} \exp \left(e_{i k}\right)}
$$
论文实验设定：
$$
\alpha_{i j}=\frac{\exp \left(\operatorname{LeakyReLU}\left(\overrightarrow{\mathbf{a}}^{T}\left[\mathbf{W} \vec{h}_{i} \| \mathbf{W} \vec{h}_{j}\right]\right)\right)}{\sum_{k \in \mathcal{N}_{i}} \exp \left(\operatorname{LeakyReLU}\left(\overrightarrow{\mathbf{a}}^{T}\left[\mathbf{W} \vec{h}_{i} \| \mathbf{W} \vec{h}_{k}\right]\right)\right)}
$$
![image-20210325091539369](https://tva1.sinaimg.cn/large/008eGmZEly1govvv9v31cj305005cdft.jpg)

图中所示：两个向量先进行线性特征变换，拼接后乘上网络权重a，经过一个非线性激活，然后softmax输出得到attention系数αij。将这种操作应用于每个邻居节点，再经过一个非线性激活函数，得到该节点的输出特征：
$$
\vec{h}_{i}^{\prime}=\sigma\left(\sum_{j \in \mathcal{N}_{i}} \alpha_{i j} \mathbf{W} \vec{h}_{j}\right)
$$
为了使模型更加稳定，本文还将多头注意力机制引入，即为每个节点使用K个独立的注意力机制，同时用K个W与原始特征向量进行线性变换，从而得到K个αij，然后将K个头的输出做一个拼接再平均，最后通过一个激活函数（通常是logistic sigmoid或softmax）得到最后的输出及结果：
$$
\vec{h}_{i}^{\prime}=\sigma\left(\frac{1}{K} \sum_{k=1}^{K} \sum_{j \in \mathcal{N}_{i}} \alpha_{i j}^{k} \mathbf{W}^{k} \vec{h}_{j}\right)
$$
![image-20210325093600031](https://tva1.sinaimg.cn/large/008eGmZEly1govwggdslwj308m05w3z5.jpg)

图中为K=3即三个头的自注意力机制。中心节点将每个邻居的多头注意力系数平均后拼接，作为这一层的输出。

本质上，GAT只是将原本的**标准化常数**替换为**使用注意力权重的邻居节点特征聚合函数**。

## GAT的优势

1. 和GCN相比：本文提出的的模型可以为节点不同的邻居按照重要程度赋予不同的权值，模型容量有了上升。通过学习到的注意力权重，模型的可解释性也得增长

2. 和GraphSAGE相比：最新的归纳学习方法Hamilton et al. (2017)是通过从每个节点的邻居中抽取固定数量的邻居节点来保持计算的一致性，这样的话在推理过程中，节点的全部邻居不能考虑进去，而GAT可以将节点的全部邻居计算进去。

## GAT复现

- [x] 运行DGL里的源代码

- [x] 解析代码封装的模型结构（从最底层的函数开始）

- [x] 用现有的包，跑通PPI数据的实验

- [x] 搞清楚dataloader的细节，根据PPI数据，自己写data loader

- 这部分dgl在PPI数据集中的data loader主要是根据data mode划分了不同的数据集合，没有做数据预处理，因此直接

- [x] 在服务器上实现PPI数据集对应在论文中的结果

### 实验说明

tranductive learning: 使用Cora, Citeceer, Pubmed数据集， two-layer GAT，acc标准为测试节点上的平均分类准确率（括号中为标准差）。以Citeceer为例：

![Screen Shot 2021-03-25 at 10.05.56 AM](https://tva1.sinaimg.cn/large/008eGmZEly1govxbv6u1zj30kk0abn11.jpg)

inductive learning: 使用PPI数据集，three-layer GAT，acc标准为对两个没有出现在训练中的测试图上的节点计算的micro F1 score。

### 实验结果对比

| Dataset  | Test Accuracy |    Rproductional Test Acc     |
| :------: | :-----------: | :---------------------------: |
|   Cora   |  84.02(0.40)  |      81.70 (200 epochs)       |
| Citeseer |  70.91(0.79)  |      70.60 (200 epochs)       |
|  Pubmed  |  78.57(0.75)  |      78.90 (200 epochs)       |
|   PPI    |    0.9836     | 0.9472 (diff: early stopping) |

 



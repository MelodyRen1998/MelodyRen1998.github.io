---
title: 2020秋季大数据挖掘与机器学习学习笔记
author: Yimeng
date: '2021-01-28'
permalink: /posts/2021/01/lecture-notes-data-mining-2020/
categories:
  - course
tags:
  - data-mining
  - lecture-notes
  - RUC
---

## LASSO问题的求解

- 问题重述
- 单变量求解
- 多变量循环坐标下降

课堂笔记：[LASSO问题的求解.pdf](/files/LASSO.pdf)

参考资料：[Lasso: Algorithms and Extensions - Yuxin Chen, Princeton University](/files/lasso_algorithm_extension.pdf)

## 凸优化和KKT条件

- 凸集和凸函数
- Lagrange 对偶
- 最优性条件

课堂笔记：[凸优化和KKT条件.pdf](/files/KKT.pdf)

## 梯度下降方法

- 次梯度与次微分
- 无约束优化问题
- 下降方法
  - 回溯直线搜索(backtracking rule)
  - 投影梯度下降(Projected Gradient Method)
  - 近端梯度下降法 (proximal gradient descent)
    - 以LASSO为例
  - 加速梯度下降(Nesterov Accelerated Gradient)
  - 交替方向乘子法(Alternating Direction Method of Multipliers, ADMM)
    - ADMM求解LASSO问题

课堂笔记：[梯度下降方法.pdf](/files/GradiantDescent.pdf)

参考资料：

[Lecture 6: Subgradient Method, September 13, Fall 2012, CMU](/files/CMU_Lecture6_subgrad.pdf)

[Proximal Gradient Descent, Ryan Tibshirani](/files/prox-grad.pdf)

[A Fast Iterative Shrinkage-Thresholding Algorithm for Linear Inverse Problems, Amir Beck and Marc Teboulle](/files/FISTA_Breck_2009.pdf)

## 熵与交叉熵

- 自信息
- 熵
- 交叉熵
- K-L 散度(一种距离表示)
- 交叉熵损失

课堂笔记：[熵与交叉熵.pdf](/files/entropy.pdf)

## 决策树

- 决策树模型与学习
- 特征选择
- 决策树的生成
  - ID3算法
  - C4.5算法
- 决策树的剪枝
- CART算法

课堂笔记：[决策树.pdf](/files/Classification_decision_tree.pdf)

## 提升方法（Boosting）

- 提升方法与 AdaBoost 算法
  - AdaBoost 算法
  - AdaBoost 的训练误差分析
  - AdaBoost 算法的解读

- 提升树
  - 提升树模型
  - 梯度提升
- 指数损失下总体的最优估计推导

课堂笔记：

[Boosting.pdf](/files/Classification_Boosting.pdf)

[指数损失下总体的最优估计.pdf](/files/LossFunc.pdf)

## GBDT, XGBoost, LightGBM

- 前向分步算法
- GBDT
- XGBoost (Extreme Gradient Boosting)
- LightGBM (理解)

课堂笔记：[其他一些boosting算法.pdf](/files/GradientMethod.pdf)

## 神经网络：反向传播算法

- 神经网络基础（参考[2019秋季DMC报告](/files/neural_network_basis_DMC.pdf)）
- 反向传播算法的推导

课堂笔记：[反向传播算法.pdf](/files/BP_algs.pdf)

## 聚类方法

- 聚类的基本概念
- 层次聚类
- k 均值聚类
- 高斯混合模型 (Gaussian Mixture Model)
  - EM算法

课堂笔记：[聚类方法.pdf](/files/clustering.pdf)

## 其他

参考资料：[SMO算法.pdf](/files/SMO.pdf)


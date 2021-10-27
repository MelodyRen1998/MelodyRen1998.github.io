---
title: 百度数据挖掘实习面经
author: Yimeng
date: '2021-03-15'
permalink: /posts/2021/03/interview-data-mining-baidu/
categories:
  - Job Hunting
tags:
  - data-mining
  - interview
  - internship
---

## 面试基本信息

3月15日下午五点通过BOSS平台约了晚上八点的面试，在消息框确认是技术面试，七点四十邮件通知技术面试链接，平台使用 https://www.showmebug.com/，面试时长1个小时。

## 流程

- 自我介绍，关于编程语言掌握和教育背景，还问了要不要读博
  - 问了scala**

- 做题（coding）
- 关于简历提问项目相关的问题
- 反向提问（面试官介绍岗位信息）

## coding

### Python 算法

1. 有一只小青蛙，每次只能跳1个或2个台阶，一共有n个台阶，小青蛙一共有多少种跳法？

   递归求解

2. 判断一棵树是不是平衡二叉树

   平衡二叉树是左子树的深度和右子树深度相差不大于1

### SQL 查询

orders表每一行表示一条消费记录，列名如下：

city_id, city_name, u_id, order_id, amount, order_date

写一个查询，得到每个城市在前一天的top10消费金额用户的信息，结果包含字段：

city_id, city_name, u_id, user_amount(消费总金额)

## 简历

1. 关于社交网络研究方面
   1. 一个社交网络结构，如何提取其中的信息
   2. 如何描述用户的重要度
2. 机器学习算法
   1. （项目）为什么回归前加了xgboost分类器（特征筛选）
   2. xgboost分类器和GBDT的异同

## 反向提问

介绍岗位主要内容，说到hadoop/spark，所以问了一下hadoop的掌握情况，mapreduce的思想

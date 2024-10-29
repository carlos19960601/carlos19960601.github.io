+++
title = 'Hands on Machine Learning With Scikit Learn, Keras, and TensorFlow 学习笔记'
date = 2024-10-29T14:35:16+08:00
draft = false
+++

# Chapter 1: The Machine Learning Landscape

机器学习分类：

**是否监督**

* 监督学习(Supervised learning)
* 非监督学习(Unsupervised learning)
* 半监督学习(Semi-supervised learning)
* 自监督学习(Self-supervised learning)
* 强化学习(Reinforcement learning)


**批量vs在线学习**

* Batch learning
* Online learning


**Instance-Based Versus Model-Based Learning**

* Instance-based learning
* Model-based learning and a typical machine learning workflow
  

机器学习的主要挑战

* 训练数据量不足(Insufficient Quantity of Training Data)
* 非代表性训练数据(Nonrepresentative Training Data)
* 劣质数据(Poor-Quality Data)
* 不相关特征(Irrelevant Features)
* 过拟合(Overfitting the Training Data)
* 欠拟合(Underfitting the Training Data)


# Chapter 2: End-to-End Machine Learning Project

通过一个预测房价的实例介绍机器学习实践流程。

前提是明确需求的情况下，现在手里有一份数据。怎么开始进行机器学习。

1. 首先了解数据，整体熟悉数据结构。
   1. panda查看数据统计
   2. matplotlib绘制图标
2. 创建测试集
   1. 分层抽样
   2. 测试集不要在训练阶段使用
   3. 抽样要随机
   4. 结果要保持一致，不要多次创建测试集后，让模型学习了所有数据
3. 看看数据分布
   1. 看看相关性(Correlations)
   2. 尝试组合特征
4. 数据预处理
   1. 缺失值补齐
   2. 处理分类数据
   3. 特征缩放
5. 选择和训练模型
   1. 尝试不同的模型
   2. 交叉验证不同模型的效果
6. 微调模型
   1. Grid Search
   2. Randomized Search
7. 分析模型和错误
8. 在测试集上评估模型
9. 上线模型

# Chapter 3: Classification

通过一个识别手写数字的实例介绍分类。

识别手写数字和预测房价不同的地方在于Performance Measures。预测房价使用是通哟RMSE(root mean squred error)。识别手写数字这里使用这种方式就会有误导。

* 混淆矩阵(Confusion Matrices)
* 准确率和召回率(Precision and Recall)
* ROC缺陷(The ROC Curve)
  
分类又可以分为多种

* Multiclass Classification
* Multilabel Classification
* Multioutput Classification

# Chapter 4: Training Models


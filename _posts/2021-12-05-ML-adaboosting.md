---
layout: post
title: "Adaboosting"
date: 2021-12-05
description: "Adaboosting"
tag: 机器学习
katex: true
---

Adaboosting 是 boosting 算法的代表之一，主要解决二分类问题。

## 思路

加法模型
$$
f(X) = \sum_{m=1}^M \alpha_m G_m(X)=\alpha_1 G_1(X)+\alpha_2 G_2(X)+ \dots +\alpha_m G_m(X)
$$
其中，$\alpha_m$由基分类器的**“分类误差率”**决定，加大【分类错误率】小的弱分类器的权值，使其在表决中起较大的作用；减小【分类错误率】大的弱分类器的权值，使其在表决中起较小的作用。

基分类器的训练样本的权重会发生变化：提高前一轮“错误分类”样本的权值，降低前一轮“正确分类”的样本权值，这样一来那些没有得到正确分类的数据，由于其权值加大，会受到后一轮分类器的更大关注。

## 算法流程

1. 初始化/ 更新当前**训练数据的权值分布**

   第一轮训练时先初始化：
   $$
   D_1=(w_{11},\dots ,w_{1i},\dots ,w_{1N}), \ w_{1i}=\frac{1}{N}, \ i=1,2,\dots , N
   $$
   其中，$D_1$表示第一轮样本的权值， N 为样本个数

2. 训练当前基分类器$G_m(X)$

   使用具有权值分布$D_m$的训练数据集学习，得到基分类器$G_m(X)$

3. 计算当前基分类器的权值$\alpha_m$

   (a) 计算当前$G_m(X)$在训练集上的**分类误差率**
   $$
   e_m=\sum_{i=1}^N P(G_m(x_i) \neq y_i) = \sum_{i=1}^N w_{mi}I(G_m(x_i) \neq y_i) = \sum_{G_m(x_i) \neq y_i} w_{mi}
   $$
   (b) 计算$G_m(X)$的系数
   $$
   \alpha_m = \frac{1}{2} \log \frac{1-e_m}{e_m} 
   $$
   这里的对数是自然对数。这个公式实现了**加大分类错误率小的基分类器的权重，减小分类错误率大的基分类器的权重**

4. 将$ \alpha_m G_m(X)$更新到加法模型f(X)中
   $$
   f(x)= \sum_{m=1}^{M} \alpha_mG_m(x)
   $$
   $f(x)$算出来是一个数不是类别，所以叫上`sign`函数转换为类别
   $$
   G(x)=sign[f(x)]=sign[ \sum_{m=1}^{M} \alpha_mG_m(x)]
   $$
   例如当 `f(x)>0, G(x) =1`

5. 判断是否满足循环退出条件

   判断条件有：循环M的次数，最后的精度等

6. 若没有退出循环条件，则会更新训练集的权重$D_m$,提高前一轮被错误分类样本的权重，让下一轮训练时更加重视这部分样本：
   $$
   D_m = (w_{m,1},\dots , w_{m,i}, \dots , w_{m,N})
   $$
   其中在第m轮循环时，样本的权重
   $$
   w_{m,i} =\frac{w_{m-1,i}}{Z_{m-1}} \exp(-\alpha_{m-1} y_i G_{m-1}(x_i)), \ i=1,2,\dots , N
   $$
   这里的$Z_{m-1}$是规范化因子（全部权重和）
   $$
   Z_{m-1} = \sum_{i=1}^N w_{m-1,i}\exp(-\alpha_{m-1} y_i G_{m-1}(x_i))
   $$
   规范化因子相当于归一化，保证每个权重在（0,1）中，使$D_m$成为一个概率分布

   对于样本权重$w_{m,i}$，可以改写为
   $$
   w_{m,i} = \begin{cases} 
   \frac{w_{m-1,i}}{Z_{m-1}} \exp(-\alpha_{m-1}) \ , & G_{m-1}(x_i) = y_i \\
   \frac{w_{m-1,i}}{Z_{m-1}} \exp(\alpha_{m-1}) \ , & G_{m-1}(x_i) \neq y_i
   \end{cases}
   $$
   所以当基分类器训练正确，则该样本的权重会减小

   

   ## 算法原理

   ### 优化问题

   adaboosting解决二分类问题
   $$
   T = \{(x_1,y_1),(x_2,y_2),\dots ,(x_N,y_N) \}
   $$
   每个样本点由实例与标记组成，y = {-1,1}

   ### 模型

   加法模型
   $$
   f(x)= \sum_{m=1}^{M} \alpha_mG_m(x)
   $$
   
   ### 最终分类器
   
   $$
   G(x) = sign[f(x)]
   $$
   
   
   
   ### 损失函数
   
   通常分类算法的损失函数有两种：交叉熵，指数损失函数。交叉熵损失函数主要针对 y ={1,0}，对于y={-1,1}
   
   使用指数损失函数。
   $$
   L(y,f(x)) = \exp[-yf(x)]
   $$
   假设y=1，当G(x) 分类正确，f(x) >0，所以y 与f(x) 同号，损失函数L = exp(负数) <=1 ，所以损失函数减小；同理当G(x) 分类错误，损失上升。
   
   将损失函数视为训练数据的权重
   $$
   w_{m,i} =\exp[-y_i f_{m-1}(x_i)]
   $$
   
    ### 优化算法
   
   前向分布算法


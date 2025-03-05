---
title: 论文随笔-CFARnet  ## 标题
author: Zane ## 作者
comments: true
date:  2025-03-05 13:53:12
description: 经典论文CFARnet的记录 ## 描述
tags:  [论文，CFAR] ## 标签
categories:
 - [论文] ## 分类
 - [深度学习]

top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg
cover: https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503051159957.png  ## 封面
---



## 介绍

本人最近受够代码了，下定决心来点不一样的，do some math（如果能在下方打赏就更好了，码文不易，希望能赞助     ~~买奶茶钱~~   买可乐钱）

恒虚警率这个约束是经典符合假设检验中的标准要求，随着科技进步，机器学习这一方法已经开始被人们使用在各个领域 

这篇文章的主要内容：

- 定义了一个可以用于贝叶斯和机器学习的CFAR检测器框架：将CFAR约束应用于贝叶斯最优检测器，可以用于任意复合假设检验

- 提出learning-based CFAR检测器的性能门限，证明CFAR-constrained Bayes检测器渐进收敛于GLRT

- 提出了CFARnet，一种深度学习的恒虚警检测

- 将CFARnet与GLRT进行性能对比

  - 相同性能下：CFARnet计算复杂度<GLRT

  - GLRT表现不佳时：CFARnet仍有很好的性能，且仍能保持恒虚警特性

    

## 检测器

### 经典检测器

**似然比检验**（Likelihood Ratio Test,LRT)[1]： 
$$
T_{LRT}(\boldsymbol{x}) = 2 \log \frac{p(\boldsymbol{x} \mid z = z_1)}{p(\boldsymbol{x} \mid z = z_0)}
$$
（此处渲染器出问题了，看图片吧）

![](http://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/202503051629084.png)


~~似然比检验是一种统计检验方法，用于比较两个统计模型的拟合优度。在这个问题中，我们有一个统计量 $ T_{LRT}(\boldsymbol{x}) $~~

~~$ p(\boldsymbol{x}; \boldsymbol z = \boldsymbol z_1) $ 是在参数 $ \boldsymbol z = \boldsymbol z_1 $ 下的似然函数值，而 $  p(\boldsymbol{x}; \boldsymbol z = \boldsymbol z_0) $ 是在参数 $ \boldsymbol z = \boldsymbol z_0 $ 下的似然函数值。这个统计量用于检验零假设 $ H_0: \boldsymbol z = \boldsymbol z_0 $ 还是对立假设 $ H_1: \boldsymbol z = \boldsymbol z_1 $~~

~~似然比检验统计量 $ T_{LRT}(\boldsymbol{x}) $ 的值越大，表示在 $ \boldsymbol z = \boldsymbol z_1 $ 下的似然函数值比在 $ \boldsymbol z = \boldsymbol z_0 $ 下的似然函数值大得多，因此我们有更强的证据拒绝零假设 $ H_0 $。~~

~~在实际应用中，我们通常会将 $T_{LRT}(\boldsymbol{x}) $ 与一个临界值进行比较，如果 $ T_{LRT}(\boldsymbol{x}) $ 大于这个临界值，我们就拒绝零假设 $ H_0 $ ,接收备择假设 $ H_1 $。~~

**广义似然比检验**(Generalized Likelihood Ratio Test,GLRT)[1]：
$$
T_{\mathrm{GLRT}}(\mathbf{x}) = 2\log \frac{\max_{\mathbf{z}\in\mathcal{Z}_1} p(\mathbf{x};\mathbf{z})}{\max_{\mathbf{z}\in\mathcal{Z}_0} p(\mathbf{x};\mathbf{z})}.
$$
GLRT是最流行的方法，是一种很接近性能门限的方法，但是也存在缺点，因为分子和分母都有max这一处理，设计大规模、非线性以及非凸优化处理，计算复杂度比较高

> GLRT is probably the most popular solution to composite hypothesis testing. It gives a simple recipe that performs well under asymptotic conditions. Its main downsides are that it is sensitive to deviations from its theoretical model, it is generally sub-optimal under finite sample settings, and it may be computationally expensive as both the numerator and denominator of the GLRT involve optimization problems that may be large-scale, non-linear and non-convex. Therefore, there is an ongoing search for flexible, robust, and low-cost alternatives.

### 贝叶斯检测器

贝叶斯检测器（Bayesian LRT）

错误概率：
$$
\text{Pr}_{\text{ERR}} = \text{Pr}(\hat{y} \neq y) = \sum_{y = 0}^{1} \int \mathbf{1}_{\hat{y} \neq y} p(\mathbf{x}, y, \mathbf{z}) d\mathbf{x} d y d\mathbf{z}
$$
最小化错误概率[2]，就能到得到
$$
T_{\text{BLRT}}(\mathbf{x}) = 2 \log \frac{p_{1}(\mathbf{x})}{p_{0}(\mathbf{x})} = 2 \log \frac{\int_{\mathbf{z} \in \mathcal{Z}_{1}} p(\mathbf{x} ; \mathbf{z}) p(\mathbf{z}) d\mathbf{z}}{\int_{\mathbf{z} \in \mathcal{Z}_{0}} p(\mathbf{x} ; \mathbf{z}) p(\mathbf{z}) d\mathbf{z}}\\
\gamma_{\text{BLRT}} = 2 \log \frac{\text{Pr}(y = 0)}{\text{Pr}(y = 1)}
$$


### 本文提出的检测器-CFAR贝叶斯(CLRT)

$$
\text{CLRT} : \left\{
\begin{array}{ll}
\min_{T,\gamma} & \Pr_{\text{ERR}}(T, \gamma) \\
\text{s.t.} & T \text{ is CFAR}
\end{array}
\right.
$$

$$
y=0: z_{r}=z_{r_0},z_n \\
y=1: z_{r}\neq z_{r_0},z_n
$$



在没有复杂的参数干扰时，BLRT和GLRT是等价的，其中， $ z_r $ ∈ ℝᵈʳ 是一个判别参数， $ z_n $ ∈ ℝᵈⁿ 是一个干扰参数。我们用 $ z_{𝑟_0} $ 和 $ z_{𝑟_1} $ 分别表示在 𝑦 = 0 和 𝑦 = 1 时  $ z_r $ 的真实值。

### 文章作者的发现：

在满足以下条件

- 数据由许多来自真实统计模型的独立同分布（i.i.d）样本组成：
  $$
  p(x;z_r,z _n)=∏_{i=1}^N p(x_i;z_r,z_n),N→∞
  $$

- 信号是弱的：
  $$
  ∥\bold z_{r_1}− \bold z_{r_0}∥= \frac {s}{\sqrt{N}}
  $$

-  未知参数的最大似然估计量（ML estimators）是统计有效的，并达到它们的渐近性能

有两个发现：
$$
T_{BLRT}(\bold x)\to T_{GLRT}(\bold x)+func(\bold z_{r_0},\bold z_n)
$$
GLRT是CLRT的一种变形[3]

## 基于学习的检测器

![image-20250305114947231](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503051149496.png)

我们决定将理论用于实际，提出一个机器学习框架，使其性能近似于上述理论

![image-20250305115004112](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503051150230.png)

![image-20250305115926745](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503051159963.png)

![image-20250305115955792](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503051159957.png)



**参考文献：**

[1] S.M. Kay, Fundamentals of statistical signal processing: Detection theory, in: Fundamentals of Statistical SI, Prentice-Hall PTR, 1998.

[2] David de la Mata-Moya, Maria Pilar Jabato-Ananes, Jaime Martin de Nicolas, Manuel Rosa-Zurera, Approximating the Neyman-Pearson detector with 2C-SVMS: application to radar detection, Signal Process. 131 (2017) 364-375.

[3] Pia Addabbo, Dario Benvenuti, Goffredo Poggi, Gastone Ciuti, Danilo Orlando, An application of artificial intelligence to adaptive radar detection using raw data, in: 2023 IEEE Radar Conference, RadarConf23, IEEE, 2023, pp. 1-6.

[4] Tzvi Diskin, Yiftach Beer, Uri Okun, Ami Wiesel,CFARnet: Deep learning for target detection with constant false alarm rate,Signal Processing,Volume223,2024,109543,ISSN 0165-1684

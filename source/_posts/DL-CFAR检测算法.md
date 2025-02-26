---
title: DL-CFAR检测算法
comments: true
date: 2025-02-25 17:54:06
tags: [雷达/声纳,信号处理]
categories:
 - [信号处理,CFAR]
 - [深度学习]
keywords: signal processing, CFAR ,detection
description: 利用深度学习进行CFAR
top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg
cover: https://www.helloimg.com/i/2025/02/26/67be7bf9d8d64.png
---

# 论文随笔-利用深度学习进行恒虚警检测

## 引言

之前已经提及，常用的CFAR一共有四种方法，但是都有一定的限制，CA-CFAR[1] 利用参考单元功率的算术平均值作为噪声水平估计值。它的一个变种，即单元平均恒定虚警率（GOCA-CFAR）[2]，可以提高原始方案的虚警率。虽然这两种方案在同质场景中表现良好，但在多目标场景中，它们的性能会因错误的噪声水平估计而下降。为了提高多目标场景下的性能，有人提出了最小单元平均 CFAR（SOCACFAR）[3]。然而，它在密集多目标场景中并不能显著提高性能。有序统计 CFAR（OS-CFAR）[4] 可以处理这类问题，但它带来了显著的计算复杂性.

## 一、复习

### 1.RDM的获取

以 FMCW 雷达为例，发射一个由 M 个啁啾（啁啾是频率随时间线性增加的正弦波）组成的帧，然后以逐个啁啾的方式将发射和接收的啁啾混合成 M 个中频（IF）信号。然后，我们在每个啁啾信号中提取 N 个中频信号样本，并使用预定的采样周期。如图所示，CCM 由这些逐个啁啾采样的级联列构成。

![RDM生成.png](https://www.helloimg.com/i/2025/02/26/67be7bf99ac06.png)

对信道系数矩阵CCM进行二维FFT，就能获得RDM图。公式如下

$$
\text{RDM}(n,m) = \left| 2\text{D FFT}(\mathbf{H})(n,m) \right|^2 \\= \left| \sum_{k=0}^{N-1} \sum_{l=0}^{M-1} (\mathbf{H})_{k,l} e^{j2\pi lm/M} e^{j2\pi kn/N} \right|^2
$$

在进行RDM处理后，会获取10log10(NM)的增益，以便于识别目标 。

### 2.CFAR

一种判断是否存在目标，而2D-CFAR不仅能获取方位信息，还能同时获取速度信息

## 二、DL-CFAR介绍

设要处理的RDM大小是NwxMw,首先将RDM截断(此处可以看作目标+噪声)，然后输入进网络，要注意的是，神经网络并不是进行目标识别，而是估计出所截断的RDM的噪声水平。

![DL模型.png](https://www.helloimg.com/i/2025/02/26/67be7bf9d8d64.png)

## 三、实现过程

### 1.数据预处理

截断处理原因：由于归一化的原因，噪声接近于0

### 2. 网络结构

如图所示，自定义的残差块+自定义的残差块+全连接层+全连接层+激活函数ReLU

![网络结构.png](https://www.helloimg.com/i/2025/02/26/67be7bf967233.png)
```python
class DLCFAR(nn.Module):
    def __init__(self):
        super(DLCFAR, self).__init__()
        self.res_block1 = ResidualBlock(1)
        self.res_block2 = ResidualBlock(1)
        self.flatten = nn.Flatten()
        self.fc1 = nn.Linear(16 * 16 * 1, 512)  # Assuming input size is (16, 16, 1)
        self.fc2 = nn.Linear(512, 256)

    def forward(self, x):
        x = self.res_block1(x)
        x = F.prelu(x)  # PReLU activation
        x = self.res_block2(x)
        x = F.prelu(x)  # PReLU activation
        x = self.flatten(x)
        x = self.fc1(x)
        x = F.relu(x)
        x = self.fc2(x)
        return x
```

该网络特点：设计神经网络时不包含任何池化层，因为希望保留 RDM的所有信息，以精确估计噪声水平。

### 3.训练方式

- 损失函数：MSE
- 优化器：Adam
- 初始学习率：0.00005
- batch_size:128
- 训练次数：500
- 训练数据集的样本数为 40000 ，验证和测试数据集的样本数均为 200000 

### 4.性能

在较高信噪比和较低信噪比的时候，性能会有所下降。下图为测试结果偏差和标准差，加粗的是该组中性能能好的。

![性能.png](https://www.helloimg.com/i/2025/02/26/67be7c0d1a430.png)

代码：（原作者[paulchen2713](https://github.com/Vanillaholic/DL_CFAR_data/commits?author=paulchen2713)）

数据生成：https://github.com/Vanillaholic/DL_CFAR_data

目标检测：https://github.com/Vanillaholic/DL_CFAR  

参考文献:

[1] C. R. Barrett, Adaptive thresholding and automatic detection.Boston, MA: Springer US, 1987, pp. 368–393. [Online]. Available:
https://doi.org/10.1007/978-1-4613-1971-912

[2] V. G. Hansen and J. H. Sawyers, “Detectability loss due to ”greatest of”selection in a cell-averaging cfar,” IEEE Trans. Aerosp. Electron. Syst.,vol. AES-16, no. 1, pp. 115–118, Jan. 1980

[3] G. V. Trunk, “Range resolution of targets using automatic detectors,”IEEE Trans. Aeros. Electron. Syst., vol. AES-14, no. 5, pp. 750–755,Sept. 1978.

[4] J. T. Rickard and G. M. Dillard, “Adaptive detection algorithms formultiple-target situations,” IEEE Trans. Aeros. Electron. Syst., vol. AES-13, no. 4, pp. 338–343, July 1977.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU3NTEyMDQ0OF19
-->
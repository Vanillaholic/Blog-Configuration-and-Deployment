---
title: 恒虚警检测器CFAR
comments: true
date: 2025-02-24 19:16:00
tags: [雷达/声纳,信号处理]
categories:  
 - [信号处理,CFAR]
keywords: signal processing, CFAR ,detection
description: 1维的恒虚警检测器CFAR 
top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg

cover: ./image/CFAR/1d-cafr.png

---
# 恒虚警检测器CFAR

## 一、概述

雷达的检测过程可用门限检测来描述。几乎所有的判断都是以接收机的输出与某个门限电平的比较为基础的，如果接收机输出的包络超过了某一设置门限，就认为出现了目标。
雷达在探测时会受到噪声、杂波和干扰的影响，因而采用固定门限进行目标检测时会产生一定的虚警，特别是当杂波背景起伏变化时虚警率会急剧上升，严重影响雷达的检测性能。因此，根据雷达杂波数据动态调整检测门限，在虚警概率保持不变的情况下实现目标检测概率最大化，这种方法称为**恒虚警率（Constant False Alarm Rate，CFAR）检测技术**。

- 虚警：在没有目标时判断为有目标（没有->有）
- 漏警：在有目标时判断为没有目标（有->没有）

## 二、CFAR检测算法

### 1.基本原理

在实际工作过程中，将信号与阈值进行比较来判断信号的有无，这个阈值的计算就用到了检测概率和虚警概率、漏警概率。对于声纳系统的来说，**阈值的选择目的在于保证最大化的检测概率以及虚警概率小于一定的量级。**

在CFAR中，检测需要一个指定的距离单元，常被成为被，称为检测单元(CUT, cell under test)，噪声功率根据临近的距离单元得到。检测的阈值为T，有如下表达式：
$$
T=aP_n
$$
P_n表示噪声功率估计，a是缩放因子，选取一个合适的因子，虚警概率就可以保持为一个常数，因此该方法称为CFAR。

![CFAR-TH.png](https://www.helloimg.com/i/2025/02/26/67be7c0d1657b.png)

阈值随着信号的噪声功率增加，以保持恒定的虚警率。当信号水平超过阈值时，会发生检测。

### 2.典型的CFAR算法

##### 2.1 均值类CFAＲ(CA-CFAＲ)算法

一维情况如下所示：

![CFAR.png](https://www.helloimg.com/i/2025/02/26/67be7c0c72f26.png)

![1d-cafr.png](https://www.helloimg.com/i/2025/02/26/67be7bf9bdf49.png)

最中间的检测单元，其次是守护单元(保护单元主要用在单目标情况下，防止目标能量泄漏到参考单元影响检测效果)，最外围是训练单元，噪声估计可以计算为：
$$
P_n = \frac{1}{2N} \sum_{m=1}^{2N} x_m
$$
其中，训练单元的长度为2N，x是训练单元中的样本[1]

> With the above cell averaging CFAR detector, assuming the data passed into the detector is from a single pulse, i.e., no pulse integration involved, the threshold factor can be written as [1]
> 使用上述单元平均 CFAR 检测器，假设传入检测器的数据来自单个脉冲，即不涉及脉冲积分，阈值因子可以写为 [1]

$$
\alpha = 2N( P_{fa}^{-1/2N} - 1)
$$
推导见[4]

```matlab
%% 距离测量 (Range Measurement)
% 将混频信号（Mix）重塑为 Nr x Nd 的矩阵
% Nr：距离维度（每个 Chirp 的采样点数）
% Nd：多普勒维度（Chirp 数量）
sig = reshape(Mix, [Nr, Nd]);

% 在距离维度 (Nr) 上对打拍频信号进行 FFT 变换
sig_fft1 = fft(sig, Nr); 

% 对 FFT 结果进行归一化处理
sig_fft1 = sig_fft1 ./ Nr;

% 取 FFT 结果的幅值（只关注频谱强度）
sig_fft1 = abs(sig_fft1);

% FFT 结果是双边谱，只保留单边谱（正频率部分）
sig_fft1 = sig_fft1(1:(Nr/2));

% 绘制距离维度的 FFT 结果
figure('Name','Range from First FFT')

% 绘制 1D FFT 输出，展示距离信息
plot(sig_fft1, "LineWidth",2);
grid on;axis ([0 200 0 0.5]);
xlabel('range');ylabel('FFT output');title('1D FFT');
```

##### 2.2 最大选择GO(Greatest Of)-CFAR &最小选择SO(Smallest Of)-CFAR

最大选择就是选取前后n个训练单元之和进行对比，选取最大值作为噪声功率水平
$$
\left\{
\begin{array}{l}
Y_1 = \sum_{i=1}^{n} X_i \\
Y_2 = \sum_{i=n+1}^{2n} X_i \\
GO: \max(Y_1, Y_2)
\end{array}
\right.
$$
最小选择就是选取最小选择
$$
\left\{
\begin{array}{l}
Y_1 = \sum_{i=1}^{n} X_i \\
Y_2 = \sum_{i=n+1}^{2n} X_i \\
SO: \min(Y_1, Y_2)
\end{array}
\right.
$$


##### 2.3 有序统计类CFAＲ(OS-CFAＲ)算法

OS(Order Statistic)CFAR的原理是先对参考单元从小到大排序
$$
 P_{fa, os} = k \binom{R}{k} \frac{\Gamma(R-k+1+T) \Gamma(k)}{\Gamma(R+1+T)} 
$$
*R*=2*n*，k  为OS-CFAR中的参数，其值的选取对算法的检测性能有较大影响[3]。

### 三、性能比较

| CFAR 类型 | 参考电平 Z          | 适用场合                                                     | 缺点                                                         |
| --------- | ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CA-CFAR   | (X+Y)/2             | 均匀杂波背景                                                 | 在杂波边缘会引起虚警率的上升；多目标环境中的检测性能较差。   |
| SO-CFAR   | min{X, Y}           | 在干扰目标位于前沿或后沿滑窗之一的多目标环境中能分辨出主目标。 | 杂波边缘和均匀杂波环境中的检测性能较差。                     |
| GO-CFAR   | max{X, Y}           | 在杂波边缘和均匀杂波环境能保持较好的检测性能。               | 多目标环境中的检测性能较差。                                 |
| OS-CFAR   | (ascend_sort{X, Y}) | 多目标检测性能较好。                                         | 依赖于参考窗内的所有样本数据，且 k 的取值直接决定了检测结果的优劣。 |

![CFAR_performance.jpg](https://www.helloimg.com/i/2025/02/26/67be7c0d00b60.jpg)

```matlab
% ------ 程序功能：四类CFAR检测算法的检测概率与SNR的关系 %
clc
clear all;
close all;

%% 参数设置
R = 24;                     % 参考单元长度
n = R/2;                    % 半滑窗长度
k = R*3/4;                  % os-cfar的参数
P_fa = 1e-6;                % 虚警概率
SNR_dB = (0:30);            % 信噪比
SNR = 10.^(SNR_dB./10);     % 信号功率与噪声功率的比值
syms T;                     % 门限因子的符号变量
%% CA-CFAR
T_CA = P_fa^(-1/R)-1;           % CA-CFAR的门限因子
Pd_CA = (1+T_CA./(1+SNR)).^(-R);    % CA-CFAR的检测概率

%% SO-CFAR、GO-CFAR
Pfa_SO = 0;
syms T
for i = 0:n-1
    Pfa_SO = Pfa_SO+2*nchoosek(n+i-1,i)*(2+T)^(-(n+i));     % SO-CFAR的虚警概率表达式
end
T1_SO = solve(Pfa_SO == P_fa, T);       % 求解出虚警概率为P_fa时对应的门限因子T
T2_SO = double(T1_SO);
T_SO = T2_SO(T2_SO == abs(T2_SO));      % SO-CFAR的门限因子

Pfa_GO = 2*(1+T)^(-n)-Pfa_SO;           % GO-CFAR的虚警概率表达式
T1_GO = solve(Pfa_GO == P_fa, T);       % 求解出虚警概率为P_fa时对应的门限因子T
T2_GO = double(T1_GO);
T_GO = T2_GO(T2_GO == abs(T2_GO));      % GO-CFAR的门限因子

Pd_SO = 0;
Pd_GO = 0;
for j = 0:n-1
    Pd_SO = Pd_SO+2*nchoosek(n+j-1,j).*(2+T_SO./(1+SNR)).^(-(n+j));     % SO-CFAR的检测概率
    Pd_GO = Pd_GO+2*nchoosek(n+j-1,j).*(2+T_GO./(1+SNR)).^(-(n+j));
end
Pd_GO = 2.*(1+T_GO./(1+SNR)).^(-n)-Pd_GO;         % GO-CFAR的检测概率

%% OS-CFAR
Pfa_OS = k*nchoosek(R,k)*gamma(R-k+1+T)*gamma(k)/gamma(R+T+1);           % OS-CFAR的虚警概率表达式
T1_OS = solve(Pfa_OS == P_fa, T);       % 求解出虚警概率为P_fa时对应的门限因子T
T2_OS = double(T1_OS);
T_OS = T2_OS(T2_OS == abs(T2_OS));      % OS-CFAR的门限因子
Pd_OS = k*nchoosek(R,k)*gamma(R-k+1+T_OS./(1+SNR))*gamma(k)./gamma(R+T_OS./(1+SNR)+1);      % OS-CFAR的检测概率

%% 画图
figure;
plot(SNR_dB,Pd_CA,'r-*');
hold on;
plot(SNR_dB,Pd_SO,'k-^');
plot(SNR_dB,Pd_GO,'b-o');
plot(SNR_dB,Pd_OS,'m-.');
grid on;
xlabel('SNR','FontName','Time New Romans','FontAngle','italic');
ylabel('P_{d}','FontName','Time New Romans','FontAngle','italic');
title(['恒虚警率 P_{fa}= ',num2str(P_fa),'，参考单元 2n= ',num2str(R)]);
legend('CA','SO','GO','OS');
```

### 参考文献

[1] Mark Richards, *Fundamentals of Radar Signal Processing*, McGraw Hill, 2005

[2] 王蓓. 基于杂波图的恒虚警处理技术研究[D]. 西安电子科技大学, 2018.

[3] Detection loss due to interfering targets in ordered statistics CFAR.

[4] P. P. Gandhi and S. A. Kassam, "Analysis of CFAR processors in nonhomogeneous background," in IEEE Transactions on Aerospace and Electronic Systems, vol. 24, no. 4, pp. 427-445, July 1988, doi: 10.1109/7.7185.
keywords: {Detectors;Radar detection;Degradation;Radar clutter;Senior members;Statistical distributions;Process design;Performance analysis;Clouds;Noise level},
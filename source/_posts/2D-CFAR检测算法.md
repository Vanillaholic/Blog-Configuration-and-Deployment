---
title: 2D-CFAR检测算法
comments: true
date: 2025-02-24 19:26:50
tags: [雷达/声纳,信号处理]
categories:  
 - [信号处理,CFAR]
keywords: signal processing, CFAR ,detection
description: 二维的恒虚警检测器CFAR ,用来探测目标移动速度和距离。
top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg
cover: ./image/CFAR/2d-cfar.png
---

## 一、内容复习

CFAR检测算法属于信号检测中的自动检测算法，在雷达信号处理中主要应用的有三种，即CA-CFAR、SO-CFAR、GO-CFAR，这三种也是初学者最常采用的算法，要求每一个雷达工程师必须掌握其基本原理，如表所示，其中OS-CFAR一般不常用。

| CFAR 类型 | 参考电平 Z          | 适用场合                                                     | 缺点                                                         |
| --------- | ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CA-CFAR   | (X+Y)/2             | 均匀杂波背景                                                 | 在杂波边缘会引起虚警率的上升；多目标环境中的检测性能较差。   |
| SO-CFAR   | min{X, Y}           | 在干扰目标位于前沿或后沿滑窗之一的多目标环境中能分辨出主目标。 | 杂波边缘和均匀杂波环境中的检测性能较差。                     |
| GO-CFAR   | max{X, Y}           | 在杂波边缘和均匀杂波环境能保持较好的检测性能。               | 多目标环境中的检测性能较差。                                 |
| OS-CFAR   | (ascend_sort{X, Y}) | 多目标检测性能较好。                                         | 依赖于参考窗内的所有样本数据，且 k 的取值直接决定了检测结果的优劣。 |

一维CFAR检测流程图如下所示

![一维CFAR检测图](./image/CFAR/1d-cafr.png)

## 二、距离多普勒矩阵（Range-Doppler Matrix，RDM)

在检测过程中，除了要知道到目标的距离信息以外，还要知道目标的速度信息，因此1D-CFAR不再满足我们的要求，而是需要采用2D-CAFR。而2D-CAFR处理的对象就是距离多普勒矩阵（Range-Doppler Matrix），RDM的形成过程如下所示

![RDM图的形成](./image/CFAR/RDM.png)

2D-CFAR是对两个维度同时检测，

![第一种2D-CFAR检测](./image/CFAR/2d-cfar.png)

还有一种方法是两次CFAR，即先对某一个维度做一次，然后又对另一个维度做一次，总共两次CFAR。比如先对速度维做一次CA-CFAR，然后对距离维做一次OS-CFAR，如图7所示。这样做的目的可以减少计算量，节约计算时间。因为当检测出具有速度的目标后，只针对动目标检测要比检测全部元素点要快速。

![第二种2D-CFAR检测](./image/CFAR/2d-cfar2.png)

## 三、二维CFAR（2D-CFAR）算法原理与仿真

前面阐述了一些概念性内容，下面是本文的核心部分。

如图所示，是2D-CFAR的原理模型。我们可以设计一个循环程序，通过在参考单元和保护单元的边缘提供边距，使 CUT 在距离多普勒图上滑动。对于每次迭代，求所有参考单元中所有信号电平的和并取平均。接下来，将 CUT （被检测单元）下的信号与此阈值进行比较。如果CUT > 阈值，则认为是目标信号，并为其分配值 1，否则认为是噪声信号，并将其置为0。（类似神经网络里的conv2D有木有？）

上述过程将生成一个阈值块，如图中绿色和红色组成的区域。因为 CUT 不能位于RDM谱矩阵的边缘，故而该阈值块小于距离多普勒图， 因此存在一部分点不会被检测到，但需要对这部分未检测到的点进行处理，关于这个问题后面会讨论。

![2D-CFAR原理](./image/CFAR/image-20250224175810961.png)

## 四、仿真

1. 生成FMCW信号

```matlab
%% FMCW 波形生成

B = c /(2*range_resolution);       %带宽
Tchirp= (5.5*2*max_range)/c;       %计算 Chirp 时间（5.5 倍往返时间）
slope = B/Tchirp;                  %FMCW斜率
disp(slope);                       % 显示斜率
fc= 1.5e3;                         %载波频率
                                                          
% 一组 Chirp 信号的数量（建议使用 2 的幂次方以便 FFT 计算多普勒频谱）
Nd=128;                            % 多普勒维度（即 Chirp 数量）
%每个chirp的采样数
Nr=1024;                           % 距离维度（即每个 Chirp 的采样点数）

% 时间向量，覆盖所有 Chirp 和每个 Chirp 内的样本
t=linspace(0,Nd*Tchirp,Nr*Nd);     % 生成总时间轴
```

2. 信号生成与移动目标仿真

```matlab
for i=1:length(t)         
    
    % *%TODO* :
    % 根据时间步长更新目标的距离 (匀速运动)
    % r(t) = r0 + v*t
    r_t(i) = target_range+ target_velocity*t(i); % 目标当前距离
    td(i) = 2*r_t(i)/c;                          % 计算信号往返时间延迟
    % TODO:
    % 为每个时间点生成发射和接收信号
    % 发射信号 Tx：FMCW 信号的基本公式
    Tx(i) = cos(2*pi*(fc*t(i) + (slope*t(i)^2)/2));

    % 接收信号 Rx：考虑了往返延迟的 FMCW 信号
    Rx(i) = cos(2*pi*(fc*(t(i)-td(i)) + (slope*(t(i)-td(i))^2)/2));
    
    % TODO:
    % 生成混频信号（打拍频信号）
    % 将发射信号和接收信号逐元素相乘
    Mix(i) = Tx(i).*Rx(i);
    
end
```

3. **距离-多普勒响应 (RDM)**

 该部分实现了 2D FFT，用于生成距离-多普勒图 (RDM)。 后续可以基于该 RDM 进行 CFAR 检测。

二维 FFT 的输出是一个包含距离和多普勒频移响应的图像，其坐标轴单位最初是FFT bin（频率单元格）。 因此，为了直观反映目标的实际距离和速度， 需要根据最大取值将坐标轴从 bin 索引转换为实际的距离 (m) 和多普勒速度 (m/s)。

```MATLAB
% 将混频信号重塑为 Nr × Nd 矩阵
Mix=reshape(Mix,[Nr,Nd]);

% 对打拍频信号执行二维 FFT（分别在距离和多普勒维度上）
sig_fft2 = fft2(Mix,Nr,Nd);


% 仅保留距离维度的单边谱
sig_fft2 = sig_fft2(1:Nr/2,1:Nd);

sig_fft2 = fftshift(sig_fft2);% 对多普勒维度做 fftshift，使零频分量居中
RDM = abs(sig_fft2);        % 计算幅度并转换为 dB 值
RDM = 10*log10(RDM) ;

% ---------------------
% 绘制距离-多普勒图 (RDM)
% ---------------------

% 定义多普勒和距离坐标轴
doppler_axis = linspace(-100,100,Nd);             % 多普勒频移轴（-100 ~ 100 m/s）
range_axis = linspace(-200,200,Nr/2)*((Nr/2)/400);% 距离轴

figure;
surf(doppler_axis,range_axis,RDM);
xlabel('doppler');ylabel('range');zlabel('RDM');title('2D FFT');
```

4. **CFAR 检测**

```matlab
%在RDM上窗口滑动

% 选择训练单元 (Training Cells) 的数量
Tr = 8; % 距离维度上的训练单元数量
Td = 4; % 多普勒维度上的训练单元数量

% 选择保护单元 (Guard Cells) 的数量，围绕 CUT 以避免信号泄漏
Gr = 4; % 距离维度上的保护单元数量
Gd = 2; % 多普勒维度上的保护单元数量

% *%TODO* :
% 设置SNR偏移量 (以dB为单位)，用于调整检测灵敏度
offset = 1.4;
% *%TODO* :
%初始化一个向量来存储训练单元每个 iteration 的噪声级
noise_level = zeros(1,1);


% *%TODO* :
% 设计一个循环，使 “CUT” 在距离 - 多普勒图上滑动，同时在边缘处为训练单元和保护单元留出边界。
% 对于每次迭代，将所有训练单元内的信号电平进行求和。为了求和，需使用 db2pow 函数将数值从对数形式转换为线性形式。
% 对所使用的所有训练单元的求和值求平均。求平均后，再使用 pow2db 函数将其转换回对数形式。
% 进一步给它加上偏移量以确定阈值。接下来，将 “CUT” 下的信号与该阈值进行比较。
% 如果 “CUT” 的电平 > 阈值，则赋予其值为 1，否则将其设为 0。


   % Use RDM[x,y] as the matrix from the output of 2D FFT for implementing
   % CFAR

RDM = RDM/max(max(RDM));
 %滑动窗口遍历整个RDM
for i = Tr+Gr+1 : Nr/2-(Gr+Tr)    % 遍历距离维度
    for j = Td+Gd+1 : Nd-(Gd+Td)  % 遍历多普勒维度

        noise_level = zeros(1,1);  % 初始化当前窗口的噪声能量

       
        for p = i-(Tr+Gr): i+ (Tr+Gr) % 遍历当前CUT周围的训练单元和保护单元
            for q = j-(Td+Gd): j+(Td+Gd)

                
                if (abs(i-p)> Gr ||abs(j-q)>Gd)% 排除保护单元，只计算训练单元的能量
                   % 将dB值转换为线性功率值
                    noise_level = noise_level+ db2pow(RDM(p,q));
                end
            end

        end
        % 平均训练单元能量并转换回dB
        threshold = pow2db(noise_level/(2*(Td+Gd+1)*2*(Tr+Gr+1)-(Gr*Gd)-1));
        threshold = threshold + offset;
        
        % 进行检测：若CUT大于阈值，标记为目标（1），否则为噪声（0）
        CUT= RDM(i,j);
        if (CUT<threshold)
            RDM(i,j)=0;
        else 
            RDM(i,j)= 1; % max_T
            disp(i);
            disp(j);
        end
        
    end
end
% 上述过程将生成一个经过阈值处理的块，该块比距离 - 多普勒图小，
% 因为 “CUT” 不能位于矩阵的边缘。因此，有少量单元不会经过阈值处理。
% 为了保持地图大小不变，将这些单元的值设为 0。

RDM(RDM~=0 & RDM~=1) = 0;
%RDM(union(1:(Tr+Gr),end-(Tr+Gr-1):end),:) = 0;  % Rows
%RDM(:,union(1:(Td+Gd),end-(Td+Gd-1):end)) = 0;  % Columns


% *%TODO* :
%绘制CFAR输出
figure('Name', 'CFAR')
surf(doppler_axis,range_axis,RDM);
colorbar;
title('offset 1.4');

```

后续内容待补充……

完整代码：https://github.com/Vanillaholic/2D-CFAR-detection-of-FMCW

(本文只是学习笔记，侵删)参考：

[1] https://mp.weixin.qq.com/s/Gs8xTeQt5dtY8jMBfZ2xYA

[2] https://mp.weixin.qq.com/s/iC18AWIRj6jOnR8SJMbpZw
---
title:  使用python进行Monte Carlo实验 ## 标题
author: Zane ## 作者
comments: true
date:  2025-03-03 13:53:12
description:  描述 ## 描述
tags:  [教学,编程] ## 标签
categories:
 - [教学,编程] ## 分类
top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg 
cover: https://i-blog.csdnimg.cn/blog_migrate/7ca8e73319822a38aa6cf5bd07f04863.png  ## 封面
---

## Monte Carlo实验

### 方法一：使用multiprocessing包

使用python进行monte carlo实验，如下图所示，有16个进程可以设置

![img](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503031157277.png)

使用模板：

```python
from multiprocessing import Pool
import argparse
def run_trial(trial_idx):
    """
	此处填写实验内容，需要在之前global变量
    """
    
    return 1 if success else 0

if __name__ =="__main__":
    total_epochs = 
    #TODO: 导入terminal参数
    parser = argparse.ArgumentParser(description='Process some arguments.')
    parser.add_argument('--snr', type=float, default=-5, help='set the signal-noise ratio')
    parser.add_argument('-o','--output', type=str, default='results.txt', help='保存结果的文件名')
    # 解析参数
    args = parser.parse_args()
    
    
	with Pool(processes=4) as pool:
        # 使用 tqdm 显示进度
        results = list(tqdm(pool.imap(run_trial, range(total_epochs)),
                            total=total_epochs, desc="Running CFAR experiments"))
    
    success_count = sum(results)
    with open(args.output, 'a') as f:
        f.write(f"虚警概率为: {P_f}\n")
        f.write(f"SNR: {args.snr}\n")
        f.write(f"蒙特卡洛实验次数: {total_epochs}\n")
        f.write(f"检测成功次数: {success_count}\n")
        f.write(f"检测成功率: {success_count / total_epochs * 100:.2f}%\n")
        f.write("---------------------------------------------------------\n")

    print(f"结果已保存到 {args.output}")
```

### 方法二：使用cupy（推荐）

这个不是默认的，需要额外安装，安装好后按如下步骤调试，即可看运算能力。

![img](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503031555578.png)

本人使用的是Nvidia-GeForce4060,运算能力是8.9，正好对应

![](https://i-blog.csdnimg.cn/blog_migrate/7ca8e73319822a38aa6cf5bd07f04863.png)

此包和numpy是镜像的，所以许多函数是通用的，直接使用即可。cupy在默认情况下，所有的数组操作都是在 GPU 上进行的。CuPy 的设计理念是直接在 GPU 上创建和操作数组，因此不需要显式地将数据从 CPU 转移到 GPU。CuPy 的数组（`cupy.ndarray`）本身就是存储在 GPU 上的，而 NumPy 的数组（`numpy.ndarray`）是存储在 CPU 上的。输出：<CUDA Device 0>（表示在 GPU 上）

![image-20250303160400374](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/img/202503031604436.png)

---
title: Resnet残差网络 ## 标题
author: Zane ## 作者
comments: true
date: 2025-03-02 13:53:12
description: 描述 ## 描述
tags: [笔记] ## 标签
categories:
 - [深度学习] ## 分类
top: ## 置顶true/false
top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg 
cover: https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/d2l/dI1pRe8l19TP1IXf.png ## 封面
---
## 思考
模型越复杂,是不是带来的好处就越多呢?
![输入图片说明](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/d2l/dI1pRe8l19TP1IXf.png)

## 结构
![输入图片说明](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/d2l/z1J3qXEVllDkzTIm.png)

残差块家族也有许多成员
![输入图片说明](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/d2l/WqbWs8L80xqNc5pu.png)

### Resnet块
![输入图片说明](https://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/d2l/d47qJCeyNyPB7i85.png)

重复多次,就得到了resnet网络
## 总结
-   残差块使得很深的网络更加容易训练
    -   甚至可以训练一千层的网络
-   残差网络对随后的深层神经网络设计产生了深远影响，无论是卷积类网络还是全连接类网络

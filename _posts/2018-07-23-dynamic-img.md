---
layout:     post
title:      Rank Pooling & Dynamic Image
subtitle:   基于Rank SVM的视频动作编码
date:       2018-08-10
author:     Oscar Zhang
header-style:   text
catalog:    true
mathjax:    true
tags:
    - Theory
    - Rank SVM
    - Dynamic Image
    - Representation Learning
    - Action Recognition
    - Computer vision
---

## 动作和轮廓

不同于单张图像，视频中的信息更加复杂，动作识别是视频中最经典的问题。这个问题并不新，轮廓、剪影是以往常用的方法，Dynamic Image最近的重要工作，值得一看。      
其将一个视频序列，编码为一张具有动作剪影的图，对动作识别有重要作用。具体效果见图：

![][3]

可以看到编码之后的图片，比较好的包括了视频序列的动作。只通过观看一张图像，就可以对视频包含的动作进行分类。

## Dynamic Image

对一个视频求其Dynamic Image：

$$dynamic\_img=\sum_{t=1}^{T}\alpha_{t}\Phi_{t}$$

$$\Phi_{t}$$是视频的第$$t$$帧，系数$$\alpha_{t}=2(T-t+1)-(T+1)(H_{T}-H_{t-1})$$，$$H_{t}=\sum_{i=1}^{t}\frac{1}{t}$$，$$H_{0}=0$$。      
总是就是视频的每一帧按照特定的权值加权求和。结果并不复杂，但其来历有深深的数学功底。下面就是一些列推导过程。        

## SVM & Rank SVM

故事要从SVM说起。标准的SVM$$F(x)=wx+b$$大家都很熟悉，是分类经常用到的工具。

![][1]

而Rand SVM是为了解决样本的排序产生：

![][2]

其认为$$F(x)$$作为的得分函数，$$x=x_{2}-x_{1}$$是对两个样本排序，用$$F(x)$$和0的大小关系说明$$x_{2}$$和$$x_{1}$$的先后关系。        
最终得到的参数$$w$$($$b$$消掉了)，就是对样本$$[x_{1}, x_{2}, \dots, x_{N}]$$排序的模型。

## Rank Pooling

可以看到，样本$$[x_{1}, x_{2}, \dots, x_{N}]$$和参数$$w$$是同维度的，如果将每一帧直接拉成一个特征作为样本，那么最后得到的$$w$$也可以还原成同长宽、通道的图像显示。这样的过程可以看作是视频中的所有帧，通过一些列操作变成一张图片，每个点可以视为做了一个复杂的pooling。        

但是仅仅如此还不够。因为一个图片，拉成特征之后的维度是相当高的（224见方的图的拉成特征后维度是150528），可行性很差。      

所以通过近似求解，得到了最终的Dynamic Image算法，只要对序列中的帧进行简单加和即可:     
Bilen H, Fernando B, Gavves E, et al. Dynamic image networks for action recognition[C]//Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2016: 3034-3042.



[1]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-dynamic-img/1.png
[2]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-dynamic-img/2.png
[3]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-dynamic-img/3.png

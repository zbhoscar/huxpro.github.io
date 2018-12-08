---
layout:     post
title:      【Draft】Video Representation Learning For Action Recognition
subtitle:   利用unlabeled data进行pre-train得到权值做fine tune，用于action recognition
date:       2018-10-14
author:     Oscar Zhang
header-style:   text
catalog:    true
mathjax:    true
tags:
    - Theory
    - Deep Learning
    - Convolutional Neural Network 
    - Semi-supervised Learning
    - Unlabeled Data
    - Representation Learning
    - Action Recognition
    - Computer vision
---

## 这是个啥问题

#### Pre-train广为采用

随着CNN在CV中的广泛应用，使得视觉问题的分类性能大幅提升，不是分类问题的也大多转化成分类问题套用CNN。在实验中普遍采用了网络预处理的步骤：针对研究的问题提出模型设计网络，对网络使用ImageNet（经典带标注的大规模图像数据集）做pre-train，然后在自己的问题中fine tune。这个操作普遍可以使训练相对于随机初值有一定的提升，比如two-stream action，它的spatial stream随机初值精度40%，而使用ImageNet pre-train之后能达到70%。

|![][1]|![][2]| 
|:---:|:---:|
|Simonyan K, Zisserman A. Two-stream convolutional networks for action recognition in videos[C]//Advances in neural information processing systems. 2014: 568-576.|Carreira J, Zisserman A. Quo vadis, action recognition? a new model and the kinetics dataset[C]//Computer Vision and Pattern Recognition (CVPR), 2017 IEEE Conference on. IEEE, 2017: 4724-4733.|

#### Why

为什么这么做有提升？普遍的解释是预训练得到的权值，包含了其所用数据集的信息。在此基础之上做fine tune，就利用了这些信息。在机器学习的范畴内，如果模型相近任务类似，自然是学习过程中用到的信息越多，结果越好。

#### 更好的pre-train？

但这样的问题是，尽管ImageNet规模很大，常用来做pre-train效果也不错，但相对于unlabeled data体量还是太小了。为了利用更多的数据，很明显也就有两大出路：

- 扩大带标注的数据集。宗旨就是不断的标，依托大量的人力。在此基础之上产生了很多为了省人力的标注数据的工作，方法各异，但不管怎样都是为了得到更大规模的带标注的数据集。      
这样做好处是pre-train和train都是分类问题，网络结构可以保持，对pre-train的权值有良好的利用，原理也更加合理。
- 利用无标注的数据。直接利用无标注的数据得到预训练权值，也是研究的另一种思路。当数据不再受标注的限制，就可以利用无限的数据来帮助模型的训练，提升效果。        
但正因为没有标记，也就意味着pre-train并不能直接用分类的模型来设计，需要一定的转化。而且得到的预训练权值，从解释性角度来讲，用来做研究问题的分类训练也是需要转化的。正如此，已有工作利用unlabeled data的效果并不好，目前为止还是直接塞ImageNet做预训练最好。

处于对CNN模型的理解，可以认为pre-train得到的权值（或其中一部分），是一种表征。

在视频动作识别问题中，对预训练有格外的依赖，表征学习也发展为一个分支。

## 


[1]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/1.png
[2]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/2.png
[3]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/3.png
[4]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/4.png
[5]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/5.png
[6]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/6.png
[7]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/7.png
[8]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/8.png
[9]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/9.png
[10]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-video-rep/10.png

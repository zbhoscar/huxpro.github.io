---
layout:     post
title:      TensorFlow常用功能
subtitle:   一些实用功能的实现方法
date:       2017-10-25
author:     Oscar Zhang
header-style: text
catalog:      true
tags:
    - Coding
    - TensorFlow
---

### sess控制
TensorFlow默认占满显卡，需要按需启动sess。

    os.environ["CUDA_VISIBLE_DEVICES"] = '1'         
    config = tf.ConfigProto()  

    # config.gpu_options.per_process_gpu_memory_fraction = 0.5
    config.gpu_options.allow_growth = True
    # allow_soft_placement=True, log_device_placement=True    

    with tf.Session(config = config) as sess:
        do something
        
### 代码升级
当遇到较老的tf代码时，需要对其进行升级。
比如caffe-TensorFlow的[代码(api 0.8)](https://github.com/ethereon/caffe-tensorflow)中:     
1. 使用python2，不需要conda caffe。要[升级](https://github.com/ethereon/caffe-tensorflow/issues/114)`caffepb`为`caffe_pb2`
2. File "/home/zbh/Desktop/caffe-tensorflow-master/kaffe/tensorflow/network.py", line 180, in concat
    return tf.concat(`concat_dim=axis`, values=inputs, name=name)         
    **新格式** tf.concat(`axis=axis`, values=inputs, name=name)
3. File "/home/zbh/Desktop/caffe-tensorflow-master/examples/imagenet/dataset.py", line 22, in process_image
    new_shape = `tf.pack`([scale, scale])       
    **新格式** `tf.stack`
4. File "/home/zbh/Desktop/caffe-tensorflow-master/examples/imagenet/dataset.py", line 23, in process_image
    img = tf.image.resize_images(img, `new_shape[0], new_shape[1]`)
   File "/home/zbh/.local/lib/python2.7/site-packages/tensorflow/python/ops/image_ops_impl.py", line 808, in resize_images
    raise ValueError('\'size\' must be a 1-D Tensor of 2 elements: '        
    **新格式** tf.image.resize_images(img, `(new_shape[0], new_shape[1])`)



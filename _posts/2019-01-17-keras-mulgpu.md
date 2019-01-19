---
layout: post
title: "使用keras搭建多GPU模型"
date: 2019-01-17
description: "介绍用keras做前端，TensorFlow做后端搭建多GPU模型"
tag: keras

---

使用keras搭建多卡模型，相比使用TensorFlow搭建多卡模型要简单很多，keras会自用将batch分为多个小batch分给不同GPU，最高支持8卡并行。但是要注意，`必须使用TensorFlow作为后端`，keras版本必须在`2.0.9`以上
# 方法
```python
import tensorflow as tf
from keras.utils import multi_gpu_model
import os
os.environ['CUDA_VISIBLE_DEVICES']='0, 1, 2, 3' #指定要使用的GPU


with tf.device('/cpu:0'):
    model = 用keras实现一个model

parallel_model = multi_gpu_model(model, gpus=8)
parallel_model.fit(x, y, epochs=20, batch_size=256)

```

# 模型保存
模型保存时不要用`parallel_model`，而是用`model`来执行`.save()`方法。不然可能会报一些错误，比如：
- TypeError: can’t pickle module objects
- TypeError: can't pickle thread.lock objects


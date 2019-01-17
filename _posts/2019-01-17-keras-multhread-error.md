---
layout: post
title: "keras在多线程环境下部署的错误：Not an element of tensor graph"
date: 2019-01-17
description: "将keras用于生产环境时，比如加入Django里时，多线程环境下报的错误"
tag: keras

---

在用Django等工具对keras模型进行部署时，因为多线程原因，模型总是不能正确执行

错误提示：
> TypeError: Cannot interpret feed_dict key as Tensor: Tensor Tensor("Placeholder:0", shape=(2004, 256), dtype=float32) is not an element of this graph

# 错误原因
恢复模型的线程和预测时的线程不一致，导致graph不一样，

# 解决办法
1.在恢复模型时保存此时的graph，
```python
self.model.load_weights(model_dir)
self.graph = tf.get_default_graph()
```

2.在预测时用这个graph作为default graph

```python
with self.graph.as_default():
    self.model.predict([x])

```


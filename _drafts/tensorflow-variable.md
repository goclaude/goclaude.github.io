---
title: tensorflow Variable
date: 2018-09-08 02:33:26
categories:
- 深度学习
tags:
- 深度学习
- tensorflow


---

# 存储节点：Variable
`tf.Variable`
代表的是一个值可以通过运行 ops 来改变的Tensor，用于保存机器学习的模型参数。
Variable的修改，在不同的多个`tf.Session`中共享。
## 创建 Variable
最好的创建方式是使用 `tf.get_variable`函数，这个函数要求传入变量变量名。使用相同的变量名调用该函数时，会使用同一个变量。
```python
# [1, 2, 3] 是指 shape，默认类型是 tf.float32，默认初始化函数 glorot_uniform_initializer
my_variable = tf.get_variable("my_variable", [1, 2, 3])
my_int_variable = tf.get_variable("my_int_variable", [1, 2, 3],
  dtype=tf.int32, initializer=tf.zeros_initializer)
```

## Variable collections
很多时候，程序的不同地方需要共享 Variable，因此提供了 collections 机制。
每个 Variable 默认被放入下面两个 collection，如果不想被共享，放入`tf.GraphKeys.LOCAL_VARIABLES`。
- `tf.GraphKeys.GLOBAL_VARIABLES`：可以在多个设备间共享
- `tf.GraphKeys.TRAINABLE_VARIABLES`：TensorFlow会计算梯度
```python
my_local = tf.get_variable("my_local", shape=(), collections=[tf.GraphKeys.LOCAL_VARIABLES])
```
- Variable也可以放入自己定义的 collection，collection无需创建。
- Variable创建完以后，也可以使用`tf.add_to_collection`函数放入 collection
```python
# 把 my_local 放入 my_collection_name
tf.add_to_collection("my_collection_name", my_local)
# 获取 my_collection_name 中的 Variable 列表
tf.get_collection("my_collection_name")
```


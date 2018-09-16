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

### Variable collections

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

### 放入指定设备

把变量放入指定的GPU设备，例如下面放入第二块GPU

```python
with tf.device("/device:GPU:1")
  v = tf.get_variable("v", [1])
```

变量一般都需要放入 ps 而不是 worker，所以TF提供了`tf.train.replica_device_setter`，会自动将变量放入ps中，例如下面

```python
cluster_spec = {
    "ps": ["ps0:2222", "ps1:2222"],
    "worker": ["worker0:2222", "worker1:2222", "worker2:2222"]}
with tf.device(tf.train.replica_device_setter(cluster=cluster_spec)):
  v = tf.get_variable("v", shape=[20, 20])
```

## 初始化变量

如果使用TF底层API，需要显式地初始化变量，初始化变量使得可以从checkpoint里读取模型。

```python
# 初始化tf.GraphKeys.GLOBAL_VARIABLES collection里的所有变量
session.run(tf.global_variables_initializer()) 
```

打印所有未初始化的变量

```python
print(session.run(tf.report_uninitialized_variables()))
```

如果要使用别的变量的值初始化，最好使用`variable.initialized_value()`来获取variable的值。

## 使用变量

当成 Tensor 来使用即可，另外有`assign, assign_add`等成员方法可以对变量赋值

## 共享变量

有两种方式共享变量

- 显式地传递`tf.Variable`对象
- 使用`tf.variable_scope`对象隐式传递`tf.Variable`对象

`tf.layers`和`tf.metrics`的实现都是通过隐式传递`tf.Variable`

#### 示例

首先定义一个卷积层

```python
def conv_relu(input, kernel_shape, bias_shape):
    # Create variable named "weights".
    weights = tf.get_variable("weights", kernel_shape,
        initializer=tf.random_normal_initializer())
    # Create variable named "biases".
    biases = tf.get_variable("biases", bias_shape,
        initializer=tf.constant_initializer(0.0))
    conv = tf.nn.conv2d(input, weights,
        strides=[1, 1, 1, 1], padding='SAME')
    return tf.nn.relu(conv + biases)
```

然后创建一个两层的CNN，两层使用各自的变量，通过`tf.variable_scope()`来区分

```python
def my_image_filter(input_images):
    with tf.variable_scope("conv1"):
        # Variables created here will be named "conv1/weights", "conv1/biases".
        relu1 = conv_relu(input_images, [5, 5, 32, 32], [32])
    with tf.variable_scope("conv2"):
        # Variables created here will be named "conv2/weights", "conv2/biases".
        return conv_relu(relu1, [5, 5, 32, 32], [32])
```

如果想复用同一个变量，则需要显式说明，例如下面用两组输入训练

```python
input1 = tf.random_normal([1,10,10,32])
input2 = tf.random_normal([1,20,20,32])
with tf.variable_scope("model"):
  output1 = my_image_filter(input1)
with tf.variable_scope("model", reuse=True):
  output2 = my_image_filter(input2)
```

此外，`variable_scope`也可以复用

```python
with tf.variable_scope("model") as scope:
  output1 = my_image_filter(input1)
with tf.variable_scope(scope, reuse=True):
  output2 = my_image_filter(input2)
```


---
title: tensorflow 数据导入
date: 2018-09-08 02:33:26
categories:
- 深度学习
tags:
- 深度学习
- tensorflow

---

## 前言

`tf.data`API可以为简单的数据创造一个复杂的数据输入管道，源源不断的送到模型训练的输入里。例如，为图片数据设计的管道，可以将分布式文件系统中的多张图片叠加、添加随机扰动到每张图片，抑或是将随即的多张图片整合到一个batch中用于训练。为文本数据设计的管道，可以XXXX（没了解过这个领域，不会翻译了，反正很NB就是了）。

`tf.data`API有下面两种抽象：

1. `tf.data.Dataset`：代表一个元素序列，每个元素包含一个或多个`Tensor`对象。对于图片数据来说，可以是一个具有图片数据和标签数据的`Tensor`对。有两种主要的创建`dataset`的方式，一种是直接通过一个或多个`Tensor`对象来创建，例如`Dataset.from_tensor_slices()`；一种是从一个或多个`Dataset`来创建，例如`Dataset.batch()`。
2. `tf.data.Iterator`：`Dataset`的迭代器，最简单的是从头到尾遍历一次。`Iterator.initializer`可以重新初始化迭代器，用于遍历不同的数据集。

## 基本机制

首先把`Dataset`给搞出来，不管是直接从`Tensor`还是从`TFRecord`之类的文件格式。

然后就可以对`Dataset`做72变，可以用`Dataset.map()`之类对单个元素操作的方法，也可以用`Dataset.batch()`之类的多元素操作方法。

最后再用各种`Iterator`来遍历。

## Dataset的数据结构

`dataset -> element -> tf.Tensor`

- 每个 element 的结构都是一样的，有一个或多个`Tensor`对象，也称为component
- `tf.DType`表示张量里的变量类型，`tf.TensorShape`表示元素的形状
---
layout: post
title: "记忆网络的结构简介"
date: 2019-01-04
description: "简单介绍一下记忆网络，弄清楚结构就行了"
tag: 网络结构

---

# 1 出发点
传统的RNN/LSTM等模型的隐藏状态或者Attention机制的`记忆存储能力太弱`,无法存储太多的信息,很容易丢失一部分语义信息,所以记忆网络通过引入外部存储来记忆信息.记忆网络的一般框架如下图所示:
![](https://upload-images.jianshu.io/upload_images/3395407-f71d48ec4d0940af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500/format/webp)
它包括四个模块:I(Input),G(Generalization),O(Output),R(Response),另外还包括一些`记忆单元`用于存储记忆.
- `Input`:输入模块,用于将文本资源(文档或这KB)和问题(question)等文本内容编码成向量.然后文本资源向量会作为Generalization模块的输入写入记忆单元中,而问题向量会作为Output模块的输入.
- `Generalization`:泛化模块,用于对记忆单元的读写,也就是更新记忆的作用.
- `Output`:输出模块,Output模块会根据Question（也会进过Input模块进行编码）对memory的内容进行权重处理，将记忆按照与Question的相关程度进行组合得到输出向量.
- `Response`:响应模块,将Output输出的向量转为用于回复的自然语言答案.

# 2 End-To-End Memory Networks
由于Memory-network的自身缺陷,不太容易使用反向传播进行训练,无法进行end-to-end的训练,所以在基础的模型之上进行了扩展,形成了可以end-to-end的模型.论文中提出了单层和多层两种架构，多层其实就是将单层网络进行stack。我们先来看一下单层模型的架构
##  2.1 单层 Memory Networks
![](https://upload-images.jianshu.io/upload_images/3395407-cfd6aca175a5484d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/720/format/webp)

这一部分看的参考[^1],总结起来就是，
1. 用`q`和`input`矩阵里的记忆单元都算一个权重，然后把这个权重作用于Output矩阵，算一个加权和，就得到了句子的表示`o`，
2. 用`o`和`u`相加，然后与`W`相乘并`softmax`得到预测的输出


## 2.2 多层MN
![](https://upload-images.jianshu.io/upload_images/3395407-6f0b5ceff74f2394.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/436/format/webp)

首先来讲，上面几层的输入就是下层o和u的和。至于各层的参数选择，论文中提出了两种方法（主要是为了减少参数量，如果每层参数都不同的话会导致参数很多难以训练）。

1. Adjacent：这种方法让相邻层之间的A=C。也就是说Ak+1=Ck，此外W等于顶层的C，B等于底层的A，这样就减少了一半的参数量。

2. Layer-wise（RNN-like)：与RNN相似，采用完全共享参数的方法，即各层之间参数均相等。Ak=...=A2=A1,Ck=...=C2=C1。由于这样会大大的减少参数量导致模型效果变差，所以提出一种改进方法，即令uk+1=Huk+ok，也就是在每一层之间加一个线性映射矩阵H。


> 感受：改了以后和检索型的qa模型很相似，计算q与每个问题的相似度，然后返回答案

# 3 Key Value Memory Network
这一部分看的参考[^2]。
Memory-network的基础模型以及可以end to end的扩展形式.但是其模型还是有很多缺陷,比如只能处理简单的文本数据,无法适用于目前主流的一些QA文本资源,比如知识库和wiki文章.
![](https://upload-images.jianshu.io/upload_images/3395407-f2d4738d636fceda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/914/format/webp)
这两个模型的主要`区别`是在于输入模块在`数据处理`上的不同,这也是Key-value Memory-network可以适用于KB和wiki数据文本资源的原因.End-to-end模型在数据处理上,只是简单的求出输入的Bow向量+位置向量作为记忆向量存储在memory slot中.而Key-value模型在求每个问题时会进行一个key hashing的预处理操作，从KB source里面选择出与之相关的记忆，然后在进行模型的训练。在这里，所有的记忆都被存储在Key-Value memory中，key负责寻址lookup，也就是对memory与Question的相关程度进行评分，而value则负责reading，也就是对记忆的值进行加权求和得到输出.


# 参考
[^1]: [记忆网络-End to End Memory Network](https://www.jianshu.com/p/e5f2b20d95ff)

[^2]: [记忆网络-Key-Value Memory Network](https://www.jianshu.com/p/419eabfff208)





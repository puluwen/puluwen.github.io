---
layout: post
title: "Facebook文本分类工具fastText介绍"
date: 2019-01-08
description: "简单介绍一下fastText工具，训练速度极快"
tag: 文本分类

---

# 1 简介
fastText是Facebook在2016年开源的一个工具，主要功能有两个：
1. 训练词向量
2. 文本分类

使用了自然语言处理和机器学习中最成功的理念，包括了`词袋（bow）`已经`n-gram` 表征语句，还有使用`subword`信息。

## 关于作者
fastText的一个作者是`Thomas Mikolov`。也正是这个人在谷歌的时候，带领团队在2012年提出了`word2vec`代替了`one-hot`编码，将词表示为一个低维连续嵌入，极大促进了NLP的发展。14年她去了脸书，然后提出了word2vec的改进版:fasttext。所以fastText和word2vec在结构上很相似。

# 2 模型架构
词向量的训练，相比word2vec来说，增加了`subword`特征。word2vec仅使用词信息，但是这样使得词之间共享的信息很少，这个问题对低频词来说更严重，如果出现`OOV`(out of vocabulary)问题，新词不能利用词库中词的信息。所以fastText增加了`subword`信息，将词做进一步拆分。同一个语料对subword的覆盖率很定是高于词的覆盖率的
> 比如：中文里，字的数量是远远低于词的数量的，同一个中文语料对字的覆盖肯定超过对词的覆盖

这时候，即便出现OOV，也能共享高频词的subword信息。
在 fasttext 中，引入了两类特征：
- n-gram

示例：who am I? n-gram设置为2，n-gram特征有，`who, who am, am, am I, I`
- n-char

每个词被看做是 n-gram字母串包。为了区分前后缀情况，"<"， ">"符号被加到了词的前后端。除了词的子串外，词本身也被包含进了 n-gram字母串包。以 where 为例，n=3 的情况下，其子串分别为 `<wh, whe, her, ere, re>，where` 。

网络结构和word2vec类似，都是一个浅层网络，与word2vec不同点在于：
- word2vec的输入只有当前词的上下文2d个词，而fastText是当前句子或者当前文本的全部词的序列
- word2vec是预测当前词的概率，fastText是预测当前文本label的概率
- fastText在预测为标签的时候使用非线性激活函数，但是在中间层没有使用非线性机会函数

cbow

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1546924894875&di=a3f89ef7f03007c7c27e41ec8fa25c2a&imgtype=jpg&src=http%3A%2F%2Fimg1.imgtn.bdimg.com%2Fit%2Fu%3D1482327881%2C2608489351%26fm%3D214%26gp%3D0.jpg)

fastText

![](https://img-blog.csdn.net/2018080722073476?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ltYWluaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

同样，fastText也使用了`Hierarchical Softmax`来加速最后一层softmax的计算。

# 3 实现
安装fastText的python版本，如果要使用最新版，可以自行编译
```shell
$ git clone https://github.com/facebookresearch/fastText.git
$ cd fastText
$ pip install .
```

也可以直接使用功能pip安装稳定版
```shel
pip install fastText
```
## 3.1 词向量训练
数据准备,中文的话，分完词后空格隔开就行：
> 联名储值卡 赠送 电子 标签 绑 银行 联名 记账 卡

代码：
```python
def train_unsupervised(input, model="skipgram", lr=0.05, dim=100, 
                   ws=5, epoch=5, minCount=5, 
                   minCountLabel=0, minn=3, 
                   maxn=6, neg=5, wordNgrams=1, 
                   loss="ns", bucket=2000000, 
                   thread=12, lrUpdateRate=100,
                   t=1e-4, label="__label__", 
                   verbose=2, pretrainedVectors=""):
  """
  训练词向量，返回模型对象
  输入数据不要包含任何标签和使用标签前缀

  @param model: 模型类型, cbow/skipgram两种
  其他参数参考train_supervised()方法
  @return model
  """
  pass
```

## 3.2 分类
数据准备，和上面一样，只是每行多一个label
> __label__greet 你好 啊

代码：
```python
def train_supervised(input, lr=0.1, dim=100, 
                   ws=5, epoch=5, minCount=1, 
                   minCountLabel=0, minn=0, 
                   maxn=0, neg=5, wordNgrams=1, 
                   loss="softmax", bucket=2000000, 
                   thread=12, lrUpdateRate=100,
                   t=1e-4, label="__label__", 
                   verbose=2, pretrainedVectors=""):
  """
  训练一个监督模型, 返回一个模型对象

  @param input: 训练数据文件路径
  @param lr:              学习率
  @param dim:             向量维度
  @param ws:              cbow模型时使用
  @param epoch:           次数
  @param minCount:        词频阈值, 小于该值在初始化时会过滤掉
  @param minCountLabel:   类别阈值，类别小于该值初始化时会过滤掉
  @param minn:            构造subword时最小char个数
  @param maxn:            构造subword时最大char个数
  @param neg:             负采样
  @param wordNgrams:      n-gram个数
  @param loss:            损失函数类型, softmax, ns: 负采样, hs: 分层softmax
  @param bucket:          词扩充大小, [A, B]: A语料中包含的词向量, B不在语料中的词向量
  @param thread:          线程个数, 每个线程处理输入数据的一段, 0号线程负责loss输出
  @param lrUpdateRate:    学习率更新
  @param t:               负采样阈值
  @param label:           类别前缀
  @param verbose:         ??
  @param pretrainedVectors: 预训练的词向量文件路径, 如果word出现在文件夹中初始化不再随机
  @return model object

  """
  pass
```

## 3.3 推荐使用fasttext封装
这个工具包对fastText进行了封装，更易于使用
安装使用`pip install fasttext`就可以
分类
```python
import fasttext
# 第一个参数是前面得到的 fasttex_train.txt ，第二个参数是将要保存模型的路径，默认会加上 .bin 
# label_prefix 就是标签或类别的起始符号
classifier = fasttext.supervised("fasttext_train.txt","fasttext.model",label_prefix = "__label__")
```
加载和测试
```python
import fasttext
# 加载模型
classifier = fasttext.load_model("fasttext.model.bin",label_prefix = "__label__")

# 测试模型 其中 fasttext_test.txt 就是测试数据，格式和 fasttext_train.txt 一样
result = classifier.test("fasttext_test.txt")
print("准确率:", result.precision)
print("回归率:", result.recall)

result = classifier.predict_proba(['如何 办 卡 '], 5)# predict_proba同时返回label和置信度，第二个参数是返回置信度前几的结果

print(result)

```
# 4 总结
fastText是一个能用浅层网络取得和深度网络相媲美的精度（还是比深层网络差点），但是分类速度极快的算法（10万数据分几类，训练时间不到1秒）。按照作者的说法“在标准的多核CPU上，能够训练10亿词级别语料库的词向量在10分钟之内，能够分类有着30万多类别的50多万句子在1分钟之内”。但是它也有自己的使用条件，它适合类别特别多的分类问题，如果类别比较少，容易过拟合。



# github地址
[文本分类方法汇总](https://github.com/brightmart/text_classification)

[fasttext的python接口地址](https://github.com/salestock/fastText.py)

[faxtText官方github地址](https://github.com/salestock/fastText.py)

# 参考
1. http://albertxiebnu.github.io/fasttext/
2. https://blog.csdn.net/qq_32023541/article/details/80839800
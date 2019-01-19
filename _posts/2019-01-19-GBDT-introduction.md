---
layout: post
title: "GBDT原理、优缺点、sklearn实现简介"
date: 2019-01-19
description: "GBDT简介"
tag: 机器学习

---

基本的结构：`Gradient Boosting + 决策树 = GBDT`

# 1 回归算法流程
![](https://upload-images.jianshu.io/upload_images/2950962-d5189de9e3e7ba05.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/594/format/webp)

GBDT算法中，用初始化的X0,y0 训练弱分类器1(即基模型1)后，将y0(真实值) - y^(预测值)=e0(残差)作为y1去训练弱分类器2(即基模型2)，依次类推，直到残差拟合到最小。

# 2 分类算法
GBDT的分类算法从思想上和GBDT的回归算法没有区别，但是由于样本输出不是连续的值，而是离散的类别，导致我们无法直接从输出类别去拟合类别输出的误差。
为了解决这个问题，主要有两个方法，一个是用`指数损失函数`，此时GBDT`退化为Adaboost算法`。另一种方法是用`类似于逻辑回归`的`对数似然损失函数`的方法。也就是说，我们用的是`类别的预测概率值`和真实概率值的差来拟合损失。

# 3 要点
- Gradient Boosting是`一种Boosting方法`，它的主要思想是，每一次建立模型，是在之前建立模型损失函数的梯度下降方向。

- 一般的梯度下降是以一个样本点(xi,yi)作为处理的粒度，w是参数，f(w;x)是目标函数，即减小损失函数L(yi,f(xi;w))，优化过程就是不断处理参数w(这里用到梯度下降)，使得损失函数L最小；GB是以一个`函数作为处理粒度`，对上一代的到的函数或者模型F(X)求梯度式，即求导，决定下降方向。

- 决策树有回归树和分类树，但是因为GBDT最终预测值是各个弱分类器结果的加权累积和，所以GBDT中用的`只有回归树`（即便GBDT调整后也可以用于分类，但是不代表GBDT的树是分类树）。



# 4 优缺点

## 4.1 优点：

- 防止过拟合。GBDT的最大好处在于，每一步的残差计算其实变相地增大了分错实例(instance)的权重，而已经分对的实例(instance)则都趋向于0。这样后面的树就能越来越专注那些前面被分错的实例(instance)。

- 它的非线性变换比较多，表达能力强，而且不需要做复杂的特征工程和特征变换。

- 可解释性强，可以自动做特征重要性排序。因此在深度学习火热以前，常作为`发现特征组合‘的有效思路。

## 4.2 缺点

Boost是一个串行过程，不好并行化，而且计算复杂度高，同时不太适合高维稀疏特征，如果feature个数太多，每一棵回归树都要耗费大量时间，（这一点上大大不如SVM）。

# 5 sklearn使用
## 5.1 分类
```python
#GBDT分类(也是使用回归树)
#导入模块
from sklearn.ensemble import GradientBoostingClassifier
from sklearn import cross_validation
from sklearn.datasets import load_iris
#获取数据
iris = load_iris()
#A创建分类器
model= GradientBoostingClassifier(n_estimators=100, learning_rate=1.0, max_depth=1, random_state=0)
#切分数据集
X_train, X_test, y_train, y_test = cross_validation.train_test_split(iris.data, iris.target, test_size=0.33, random_state=42)
#训练
model.fit(X_train,y_train)
#预测
predicted= model.predict(X_test)
print(float(predicted.shape[0]-sum((predicted-y_test)*(predicted-y_test)))/predicted.shape[0])
#5折cv验证
scores = cross_validation.cross_val_score(model, iris.data, iris.target, cv=5)
print scores
```
参数说明：
```python
GradientBoostingClassifier(
loss='deviance', ##损失函数默认deviance deviance具有概率输出的分类的偏差
n_estimators=100, ##默认100 回归树个数 弱学习器个数
learning_rate=0.1, ##默认0.1学习速率/步长0.0-1.0的超参数 每个树学习前一个树的残差的步长
max_depth=3, ## 默认值为3每个回归树的深度 控制树的大小 也可用叶节点的数量max leaf nodes控制
subsample=1, ##树生成时对样本采样 选择子样本<1.0导致方差的减少和偏差的增加
min_samples_split=2, ##生成子节点所需的最小样本数 如果是浮点数代表是百分比
min_samples_leaf=1, ##叶节点所需的最小样本数 如果是浮点数代表是百分比
max_features=None, ##在寻找最佳分割点要考虑的特征数量auto全选/sqrt开方/log2对数/None全选/int自定义几个/float百分比
max_leaf_nodes=None, ##叶节点的数量 None不限数量
min_impurity_split=1e-7, ##停止分裂叶子节点的阈值
verbose=0, ##打印输出 大于1打印每棵树的进度和性能
warm_start=False, ##True在前面基础上增量训练(重设参数减少训练次数) False默认擦除重新训练
random_state=0 ##随机种子-方便重现
)
```

## 5.2 回归
```python
##GBDT回归 常调n_estimators树的个数max_depth每颗树深度
from sklearn.ensemble import GradientBoostingRegressor
clf = GradientBoostingRegressor(
  loss='ls'
, learning_rate=0.1
, n_estimators=100
, subsample=1
, min_samples_split=2
, min_samples_leaf=1
, max_depth=3
, init=None
, random_state=None
, max_features=None
, alpha=0.9
, verbose=0
, max_leaf_nodes=None
, warm_start=False
)
clf.fit(train_x, train_y)
test_y= clf.predict(test_x)

```

参数说明：
```python
GradientBoostingRegressor(
loss='ls', ##默认ls损失函数'ls'是指最小二乘回归lad'（最小绝对偏差）'huber'是两者的组合
n_estimators=100, ##默认100 回归树个数 弱学习器个数
learning_rate=0.1, ##默认0.1学习速率/步长0.0-1.0的超参数 每个树学习前一个树的残差的步长
max_depth=3, ## 默认值为3每个回归树的深度 控制树的大小 也可用叶节点的数量max leaf nodes控制
subsample=1, ##用于拟合个别基础学习器的样本分数 选择子样本<1.0导致方差的减少和偏差的增加
min_samples_split=2, ##生成子节点所需的最小样本数 如果是浮点数代表是百分比
min_samples_leaf=1, ##叶节点所需的最小样本数 如果是浮点数代表是百分比
max_features=None, ##在寻找最佳分割点要考虑的特征数量auto全选/sqrt开方/log2对数/None全选/int自定义几个/float百分比
max_leaf_nodes=None, ##叶节点的数量 None不限数量
min_impurity_split=1e-7, ##停止分裂叶子节点的阈值
verbose=0, ##打印输出 大于1打印每棵树的进度和性能
warm_start=False, ##True在前面基础上增量训练 False默认擦除重新训练 增加树
random_state=0 ##随机种子-方便重现
)
```

# 6 参考
1. https://blog.csdn.net/q383700092/article/details/53744277
2. https://www.jianshu.com/p/de6b4ae908bf
3. https://www.cnblogs.com/pinard/p/6140514.html
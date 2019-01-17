---
layout: post
title: "Rasa的安装问题"
date: 2018-12-27
description: "介绍自己在安装Rasa的过程中遇到的坑"
tag: 问答系统

---

# ubuntu上安装
使用命名
```
pip install rasa_core==0.9.8
```
安装rasa_core会自动安装对应版本的rasa_nlu。还需要安装如下包
```
pip install jieba
pip install -U scikit-learn sklearn-crfsuite
pip install git+https://github.com/mit-nlp/MITIE.git
```

# 在windows上安装
直接使用如下命名安装会报**没有C++编译器**的问题
```
pip install rasa_core
```
在Stack Overflow上查找原因，结果是rasa_nlu时，默认是安装
```
requirements_bare.txt
```
而安装这个包会安装**twisted**这个包，而这个包需要编译器编译，但是可以从[python工具包网站](http://www.lfd.uci.edu/~gohlke/pythonlibs/)（在浏览器中已收藏），直接下载**别人已经编译好的版本**。
然后使用如下命名就可以成功安装
```
pip install ./Twisted-18.7.0-cp36-cp36m-win_amd64.whl
pip install rasa_nlu==0.12.3
```
`尽量不要在windows上安装Rasa`，因为mitie的编译问题弄了几天都没有弄好，安装`linux虚拟机`是一个更好的选择，`Mac系统`也行，没有mac的`装黑苹果`（我就是，哈哈）也行。

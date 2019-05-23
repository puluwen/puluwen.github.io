---
layout: post
title: "python读取Excel常用包，可插入删除一行一列"
date: 2019-05-06
description: "介绍python读取Excel常用的包，包含插入如何一行一列"
tag: 工具

---

# 1.功能
`SPTAG`是微软在2019年2月左右开源的一个用于向量搜索的工具。[官方网站](https://github.com/microsoft/SPTAG)。
搜索速度快，200万数据，向量768维度情景下，搜索一速度为5ms。
本博客记录我使用SPTAG的一些总结，包含SPTAG安装，与使用时遇到的问题。

# 2 安装
SPTAG 的github上说只要swig、make、boost三个库，但是在装boost的时候还提示说需要intel的TBB。
具体版本为：
```
swig >= 3.0
cmake >= 3.12.0
boost >= 1.67.0
tbb >= 4.2
```

## 2.1 Ubuntu和Deppin
- 安装swig：https://blog.csdn.net/zhangkzz/article/details/88555830
- 安装TBB：https://blog.csdn.net/u010793236/article/details/74010571（Mac也适用），TBB[下载](https://github.com/01org/tbb/releases)的时候要下载源码，重新编译
- cmake：https://www.linuxidc.com/Linux/2018-09/154165.htm
- boost：https://www.jianshu.com/p/b280a9f90b05

安装好后，如果不能使用boost的库：https://blog.csdn.net/zx7415963/article/details/46845161
> Linux查看boost版本 dpkg -S /usr/include/boost/version.hpp

SPTAG：如果要使用python3编译，则要修改`Pythonwrapper`目录下的`CMakeList.txt`文件，调换python2和python3检查的顺序

```shell
find_package(Python3 COMPONENTS Development)
if (Python3_FOUND)
        include_directories (${Python3_INCLUDE_DIRS})
        link_directories (${Python3_LIBRARY_DIRS})
        message (STATUS "Found Python.")
        message (STATUS "Include Path: ${Python3_INCLUDE_DIRS}")
        message (STATUS "Library Path: ${Python3_LIBRARIES}")
        set (Python_LIBRARIES ${Python3_LIBRARIES})
else()
    message (STATUS "Could not find Python 3!")
    find_package(Python2 COMPONENTS Development)
    if (Python2_FOUND)    
        include_directories (${Python2_INCLUDE_DIRS})
        link_directories (${Python2_LIBRARY_DIRS})
        message (STATUS "Found Python.")
        message (STATUS "Include Path: ${Python2_INCLUDE_DIRS}")
        message (STATUS "Library Path: ${Python2_LIBRARIES}")
        set (Python_LIBRARIES ${Python2_LIBRARIES})
    else ()
        message (FATAL_ERROR "Could not find python2 or python3!")
    endif()
endif()
```

## 2.2 Mac
在Mac上我最终没有安装成功。SPTAG需要`g++编译器`，虽然我用`g++`替换了Mac的`clang编译器`，最后还是没有安装成功。
- 安装swig：http://blog.sina.com.cn/s/blog_4c191f7a0102z47f.html
- 安装TBB：同Ubuntu，但是想用gcc编译的话，要在Makefile文件里开头加compiler=gcc

安装SPTAG失败：最后失败在这里一步，不能生成dylib连接文件，不晓得是tbb什么问题，换成gcc编译的tbb还是报相同的问题
<img src='image/sptag-mac-install-error.png'>

# 3.原理介绍
SPTAG数据结构支持KDT和BKT，距离度量支持L2和cos距离。
- KDT就是k-d树的多维扩展，https://blog.csdn.net/xbmatrix/article/details/63683614，李航的《统计学习方法上也有》
- BKT:全称是 balanced k-means tree，目前还未找到资料

>注意：SPTAG目前（2019年5月23日）支持的数据类型包括Int8, Int16 and Float (float32)，还不支持float64,。

# 4.实验
## 4.1实验记录
向量由百度BERT、ERNIE得到，维度768维，数据类型float32
使用1w数据，在服务器1（AMD A8 5600k双核四线程， 32G内存）上，线程设置4，用时12分钟
使用1w数据，在服务器2（Intel 志强E5 1660 v2 6核12线程， 32G内存）上，线程设置为12，用时275秒
使用200w数据，报错，分别输入第一个100w和第二个100w都没问题，不会直接报错（猜想一次输入的`数据内存占用不能超过4GB`，证实：依据类型，维度，计算得到4GB除4除768=1398101.333，如果输入1398101行数据不报错，输入1398102报错）
但是又发现直接输入100万数据调用add函数时，并没有启用多线程，而且第一个100万调用build函数时，跑了35个小时还没跑完，1万1万数据的add，16个小时都加到70万了
# 测试结果
百度ERNIE CPU版本，运行10次，平均用时183ms，GPU版本，运行10次，平均用时144ms
sptag，每个问题返回1个答案时，搜索10000次，平均用时5ms，每个问题返回3个答案时，平均用时6.3ms


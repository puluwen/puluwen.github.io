---
layout: post
title: "Latex投稿时一些方法总结(持续更新) "
date: 2019-09-07
description: 总结在用Latex写会议论文和期刊论文中遇到的问题和方法
tag: 投稿

---

# 1.添加通信作者信封图标
在latex最上面增加`\usepackage[misc]{ifsym}`宏包，然后在用的时候填`\Letter`

# 2.改图表和上下文的间隙
一般会议对论文页数有限制，如果内容很多规定页数不够写，可以通过调整图表的大小、上下文间距来微调，获得一些空间。当然这样获得的空间有限
```
\vspace{-0.8cm}  %调整图片与上文的垂直距离
\setlength{\abovecaptionskip}{-0.2cm}   %调整图片标题与图距离
\setlength{\belowcaptionskip}{-1cm}   %调整图片标题与下文距离
```

# 3.设置引用颜色
有时候官方提供的模板里，引用没有颜色，非常不美观，所以需要手动加。<br>
可以根据自己的需要，给不同类型引用设置不同颜色。
```
 \usepackage[colorlinks,linkcolor=blue,anchorcolor=blue,linktocpage=true,urlcolor=blue,citecolor=blue]{hyperref}
```

# 4.一个作者多个单位
如果用会议/期刊模板里提供的方法一直搞不出单个作者多单位，可以用` \textsuperscript{1} `模拟多单位时作者名右上角角标，然后用纯文本填写作者单位 <br>
参考：https://blog.csdn.net/qq997843911/article/details/99779343
```latex
\usepackage[misc]{ifsym}

\author{XY Yin \textsuperscript{1} \and
        Boss Fan \textsuperscript{1,*} \and
        Xu Yang \textsuperscript{1} \and
        Shiguang Qiu\textsuperscript{2}
}

\institute{
\Letter Boss Fan\\
\email{fan@sjtu.edu.cn}\\     
Tel.: +86-21-34206291\\     
\at
 {1} Institute of Intelligent Manufacturing and Information Engineering, Shanghai Jiao Tong University, Shanghai, 200240, China.
 \at
 {2} Chengdu Aircraft Industry (Group) Co. Ltd. of AVIC, Chengdu, 610092, China.\\
}

```


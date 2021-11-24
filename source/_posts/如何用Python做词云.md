---
title: 如何用Python做词云
top: false
cover: false
toc: true
mathjax: false
categories:
  - Python
tags:
  - 词云
abbrlink: 11496
date: 2017-09-03 22:48:26
author:
img:
coverImg:
password:
summary:
---




**１．词云是什么?想必大家都见过这种图片，这就是词云啦**
        

![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/7f8771b93f27a485fe9c4803cdf0e086.png)
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/8f6b223da31418d3b91a8c92b4751cd5.png)


```
“词云”这个概念由美国西北大学新闻学副教授、新媒体专业主任里奇·戈登（Rich Gordon）于近日提出。戈登做过编辑、记者，曾担任迈阿密先驱报（Miami Herald）新媒体版的主任。他一直很关注网络内容发布的最新形式——即那些只有互联网可以采用而报纸、广播、电视等其它媒体都望尘莫及的传播方式。通常，这些最新的、最适合网络的传播方式，也是最好的传播方式。 因此，“词云”就是对网络文本中出现频率较高的“关键词”予以视觉上的突出，形成“关键词云层”或“关键词渲染”，从而过滤掉大量的文本信息，使浏览网页者只要一眼扫过文本就可以领略文本的主旨。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　－－－－百度百科
```

**２ . 那如何做词云呢，这些词云是怎么生成的呢**

		现在，我们用Python这门非常热门的编程语言来做词云，如果你之前没有编程基础，没关系。从零开始，意味着我会教你如何安装Python运行环境，一步步完成词云图。希望你不要限于浏览，而是亲自动手尝试一番。

**３．环境的安装**


要使用Python，我们就需要安装Python的运行环境，如果你和我一样使用的是Ubuntu系统，那么的你的系统就已经安装了Python2.7和Python3.6的运行环境，在终端下输入python2.7或python3即可启动。

![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/0d95278fac9eedbea22eca44609a72e7.png)

但这只是一个运行环境，我们知道Python是一门非常强大的语言，拥有非常多的库。所有我们要做词云，也需要安装一些库.那么我们最好是安装一个工具包，这样我们需要的库，或者扩展包都包含了，不需要我们在安装上花费太多的时间。

那么我推荐，也是业内非常推荐的一款套装，他就是大名鼎鼎的Anaconda
官方下载地址：https://www.anaconda.com/download/

![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/377a8b7239907e24075d48b0d674ae18.png)

但是这个地址下载速度实在是慢得让人抓狂，所有我给出这个下载地址
清华大学开源软件镜像站：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/
按照对应的系统版本下载即可。

这里有一个版本选择的问题我们是选择Python2.7还是Python3呢
我推荐大家选Python3,也就是Anaconda3毕竟长江后浪推前浪

![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/327b4c6f3912f6a76fa7ba0c10c45278.png)

我的是Ubuntu，下载好的是一个以.sh结尾的脚步文件。
打开终端，输入

```
bash Anaconda3-4.4.0-Linux-x86_64.sh
```
按提示输入回车，所有要求选择yes/no的都选择yes
骚等片刻，Anaconda就安装好了。


接着我们打开终端，输入

```
mkdir ciyun                  //创建一个专用的目录（个人喜好）
cd ciyun
pip install wordcloud　　　　　//安装词云wordcloud扩展包，做词云用的
//过程略 ....
pip list                      //输出的结果下有wordcloud
```


一路下载安装，完成。如果没有报错，并且在恭喜你环境就配置好了。非常简单是不是.


**４．开始动手做词云**

在开始之前，我们还需要分析的对象，也就是文本。因为中文的构成毕竟复杂，我们先选择英文文本
我这次选择的是马丁路德金的我有一个梦想演讲搞，大家可以去搜索一下。把这个txt的文本放在我们创建的ciyun目录下。我取名为dream_En.txt

在终端输入

```
jupyter notebook      //自动打开一个浏览器
```

然后切换到我们创建的ciyun目录，点击右上角的NEW,创建一个编辑器，名字随意，有些不用名字.输入一下代码

```
file = open('dream_En.txt')　　　//打开文本
text = file.read()              //读取文本
text　　　　　　　                 //输出文本　　　按Shift+Enter执行代码
```
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/8492ec22c5f6537f687c58733d072ef4.png)

到这里说明我们的数据没有问题，接着我们需要使用wordcloud对文本进行分析

```
from wordcloud import WordCloud
wordcloud = WordCloud().generate(mytext)　　//如果出现警告，忽略，不影响的
```
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/50058c32bbbaa948ab0ca54dc4459794.png)

把text用＃注释掉，防止干扰。但是到这一步却没有输出，但词云其实已经分析完成了，只是没有输出

注意：如果你在这一步报了一个错，比如

```
ImportError:cannot import name wordcloud   //类似字眼的
```
那么是你的wordcloud没有安装好，回到终端检查一下。

```
pip list   　　　　　　　　　//看看输出结果有没有wordcloud,如果没有，请继续
pip install wordcloud    　//安装
```
接着，

```
%pylab inline
import matplotlib.pyplot as plt
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")　　　　　　//忽略警告
```
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/2e7bcc84f0775791f7e74d352f16b83a.png)

是不是很激动，一张英文词云就这样做好了．简单吧！

**５.总结**

wordcloud这个扩展包的功能非常多，大家发现，做出的词云与本文开头的还是有一些差距的，那么在后续的文章中我会一一讲解。慢慢挖掘wordcloud的高级特性。


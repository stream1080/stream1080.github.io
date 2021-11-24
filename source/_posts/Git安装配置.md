---
title: Git安装配置
top: false
cover: false
toc: true
mathjax: false
abbrlink: 304
date: 2021-01-31 22:37:14
author:
img:
coverImg:
password:
summary:
categories:
  - 工具
tags:
  - Git
---

# Git使用指南
## 安装Git
安装很简单，直接在网上搜索Git，选择合适的平台和版本，下载安装包。一路回车安装完成即可，没有什么难度。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130213233579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70#pic_center)
## 注册GitHub或者Gitee

```javascript
GitHub：https://github.com/
gitee：https://gitee.com/
```

注册很简单，直接在官网注册，填写账户和邮箱信息，中间需要验证邮箱信息。
github和gitee两者的功能基本一直，国内访问 Github 速度比较慢，很影响我们的使用。
如果你希望体验到 Git 飞一般的速度，可以使用国内的 Git 托管服务——Gitee（gitee.com）。

Gitee 提供免费的 Git 仓库，还集成了代码质量检测、项目演示等功能。对于团队协作开发，Gitee 还提供了项目管理、代码托管、文档管理的服务，5 人以下小团队免费。

## 上传Git公钥
由于你的本地 Git 仓库和 GitHub 仓库之间的传输是通过SSH加密的，所以我们需要配置验证信息：
打开本地安装的Git，使用以下命令生成 SSH Key：

```javascript
$ ssh-keygen -t rsa -C "youremail@example.com"
```
youremail@example.com是你注册Github或者Gitee的邮箱。
然后到C盘用户根目录下的.ssh文件下即可找到刚刚生成的公钥和私钥
目录：C:\Users\用户名\.ssh
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130214021521.png#pic_center)
id_rsa：生成的私钥，这个不需要上传
id_rsa.pub：这是生成的公钥，外面需要把里面的内容复制到Github或者Gitee上面
下面以Gitee为例：点击gitee的用户头像，点击设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130214446464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70#pic_center)
找到左边一栏的SSH公钥，标题可以随便写，只是用来做标记的。
公钥栏就把刚刚id_rsq.pub文件里面的内容粘贴上去即可。
点击确认后需要验证用户登录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130214540395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70#pic_center)
验证结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130215311539.png)

## 创建仓库
1. 以gitee为例，在首页上面点击“+”，新建仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131120546636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
2. 输入仓库名称和仓库路径，也可以完善其他信息，比如，仓库介绍，开源协议，语言等。之后点击创建
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131121036282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
3. 创建完成后我们可以看到gitee给出的基本配置步骤，我们后面就按照这个步骤来配置仓库和提交项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131123202838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
4. 配置过程我们需要使用到一个SSH地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131123321311.png)



## 配置仓库
1. 在电脑上你的一个存放项目的目录中，右键鼠标点击Git Bash Here，配置仓库的用户名和邮箱地址，在控制台中输入以下命令：

```javascript
git config --global user.name "stream1080"
git config --global user.email "youemail@examplemail.com"
```
2. 在本地创建本地仓库

```javascript
//创建test目录文件
mkdir test
//进入test目录
cd test
//初始化仓库，让test，目录变成一个git仓库
git init
//创建README.md文件
touch README.md
//将README.md文件加入到暂存区
git add README.md
//设置提交标题，可以是修改的内容标题
git commit -m "first commit"
//提交到远程仓库Gitee
git remote add origin git@gitee.com:stream1080/test.git
git push -u origin master
```
3. 设置演示截图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131124635579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
4. 提交成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131124845711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
5. 检查远程仓库gitee
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131124917267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## 二次提交
创建好仓库后，我们常常需要多次修改代码，完善后再次提交
这次我们演示一下修改后如何二次提交
1. 修改README.md的内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131130531679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
2. 提交到远程仓库

```javascript
//查看项目中哪些文件发生了变化
git status 

//将所有变更文件添加进来
git add . 

//这个时候文件都变成了  modified:   README.md
git status 

//标记为第二次提交
git commit -m '第二次提交'

//把本地的推送到远程的分支上面即可
git push 

//查看结果，已经没用变化的文件了
git status 
 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131131959342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)




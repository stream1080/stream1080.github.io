---
title: Git之版本回退和分支合并
top: false
cover: false
toc: true
mathjax: false
abbrlink: 39129
date: 2021-06-16 22:42:17
author:
img:
coverImg:
password:
summary:
categories:
  - 工具
tags:
  - 版本控制
  - Git
---

# 版本回退
- 有时候开发一个功能，发现思路不对，需要回退到某个版本。
 - 使用git进行版本控制，就可以随意回退到任意版本
 - 这种操作叫 回滚
## git -log
- 该命令显示从最近到最远的提交日志。
- 每一次提交都有对应的 commit id 和 commit message。
- 使用  --pretty=oneline 参数，显示更清晰
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616101619746.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

## git reset --hard id
- 根据 id 回退到指定的版本；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616101840932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## git push origin HEAD --force
- 推送到远程仓库，让远程仓库和本地仓库保存版本一致
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616102154647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616102241485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## git -reflog
- 查看操作命令历史
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616102406978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## 撤销操作
- 如果突然不想回退了，可以找到需要回退的Id
- 按照上面的方法，使用git reset --hard id命令
- 就可以又回到之前的版本了


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616102823374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021061610280945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
# 合并分支
## git branch
- 列出你所有的分支、创建新分支、删除分支及重命名分支。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210616103444346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

## git checkout
- 切换分支，或者检出内容到工作目录。

![切换到master主分支](https://img-blog.csdnimg.cn/2021061610350565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

## git pull origin master
- 建议每次操作前，都将远程代码pull下来
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021061610363877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

## git merge dev
- 合并一个或者多个分支到你已经检出的分支中。 然后它将当前分支指针移动到合并结果上。
- 把dev分支的代码合并到master上
- 我这里已经是一致了的
![在这里插入图片描述](https://img-blog.csdnimg.cn/202106161039128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)



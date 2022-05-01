---
title: MySQL的执行计划(一)
cover: false
top: false
categories:
  - 数据库
tags:
  - MySQL
  - 执行计划
abbrlink: 53351
date: 2021-10-05 22:37:17
summary:
---
## 什么是执行计划
执行计划，就是一条SQL语句，在数据库中实际执行的时候，一步步的分别都做了什么事情

EXPLAIN命令是查看查询优化器是如何决定执行查询的主要方法，从它的查询结果中我们可以知道：
- 一个SQL语句每一步是如何执行的；
- 都做了哪些事，分为哪几步；
- 有没有用到索引；
- 哪些字段用到了什么样的索引，是否有一些可优化的地方等。

查看执行计划，只需在查询中的SELECT关键字之前增加EXPLAIN即可

```sql
语法：EXPLAIN + SELECT查询语句
示例：EXPLAIN SELECT * FROM `user`
```

使用EXPLAIN时，会返回执行计划中每一步的信息，它会返回一行或多行信息，显示出执行计划中的每一部分和执行的次序。

```sql
EXPLAIN SELECT * FROM `user`
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/65163a07c98048638d4dfab62c0045a8.png)
## 执行计划中的列
![在这里插入图片描述](https://img-blog.csdnimg.cn/bb0128492f804b059e162e8af194fd0a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_15,color_FFFFFF,t_70,g_se,x_16)
### Id
id是一个编号，用于标识SELECT查询的序列号，表示执行SQL查询过程中SELECT子句或操作表的顺序。

- 如果在SQL语句中没有子查询或关联查询，那么id列为1
- 否则，内层的SELECT语句一般会顺序编号；

Id列可能会存在三种情况，以下一一列举：
#### Id相同
只有普通的查询，没有子查询，则Id相同为1
```sql
EXPLAIN SELECT * FROM `user`,`comment` WHERE `comment`.user_id = `user`.id
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/057a89f6b571498fbefb5f821a4145a2.png)
#### Id不同
存在子查询，id的序号会递增，id值越大优先级越高，越先被执行。

```sql
EXPLAIN SELECT * FROM `user` WHERE id = 
(SELECT user_id FROM `comment` WHERE entity_type = 2 LIMIT 1)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/310ff579697e43a497a2a7970fc28bb4.png)

#### Id相同和不同
id如果相同，认为是一组，从从上往下执行。
在所有组中，id值越大，优先级越高，越先执行。
```sql
EXPLAIN SELECT * FROM `user` WHERE id IN 
(SELECT user_id FROM `comment` WHERE entity_type = 2 )
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/03a08fc0cd824c5181da896aabfb1815.png)
### select_type列
select_type列表示对应行的查询类型，是简单查询还是复杂查询
- 主要用于区分普通查询、联合查询、子查询等复杂的查询。 
![在这里插入图片描述](https://img-blog.csdnimg.cn/522f97c08a8d423a881a4973d547b41d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
### table
table列表示对应行正在执行的哪张表，指代对应表名
- 如果SQL中定义了别名，则会显示该表的别名

### partitions
- 查询涉及到的分区。

下一篇

[MySQL执行计划(二)](https://blog.csdn.net/upstream480/article/details/120615700)
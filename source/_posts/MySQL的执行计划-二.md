---
title: MySQL的执行计划(二)
cover: false
top: false
categories:
  - 数据库
tags:
  - MySQL
  - 执行计划
abbrlink: 4291
date: 2021-10-06 22:37:23
summary:
---
书接上回：[MySQL执行计划(一)](https://blog.csdn.net/upstream480/article/details/120615005)

## 执行计划中的列
### type
type列指代访问类型，是MySQL决定如何查找表中的行。

是SQL查询优化中一个很重要的指标，拥有很多值，依次从最差到最优：

#### ALL 
全表扫描，性能最差，在写SQL时尽量避免此种情况的出现，也就是避免是

```sql
SELECT * 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e83044a80d254d0fb5fc9357bb7e521c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
#### index
全索引查询，和全表查询的ALL类似，但是扫描表时按索引次序进行（遍历索引树），而不是按行扫描

index与ALL虽然都是读全表，但index是从索引中读取，而ALL是从硬盘读取。
显然，index性能上明显优于ALL，所有，合理的添加索引将有助于提升性能。

举例如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2fabd07cdb3d4966b999143de1a18214.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
#### range
只查询给定范围的行，使用一个索引来选择行。
- key列显示使用了那个索引；
- 一般就是在where语句中出现了bettween、<、>、in等的查询；
- 这种索引列上的范围扫描比全索引扫描index要好。![在这里插入图片描述](https://img-blog.csdnimg.cn/623618fb584d447e96dd743f8d47c7ee.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
####  ref 
非唯一性索引扫描，返回匹配某个单独值的所有行。本质是也是一种索引访问，它返回所有匹配某个单独值的行，然而它可能会找到多个符合条件的行，所以它属于查找和扫描的混合体。

此类型只有当使用非唯一索引或者唯一索引的非唯一性前缀时，才会发生。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d76d82b62b14e6dbdb453d2a7e561c0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)

#### eq_ref 
- 唯一索引扫描
- 主要用于主键或唯一索引扫描。

#### const  
- 通过索引一次就能找到，const用于比较primary key 或者unique索引。
- 因为只需匹配一行数据，所有很快。
- 将主键置于where列表中，mysql就能将该查询转换为一个const。
![在这里插入图片描述](https://img-blog.csdnimg.cn/accbeada941a4b16ada04d80985d3232.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)

#### system
- 表只有一行记录，这是const类型的特例，比较少见。


下一篇
[MySQL执行计划(三)](https://blog.csdn.net/upstream480/article/details/120616953)
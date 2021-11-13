---
title: MySQL的执行计划(三)
cover: false
top: false
date: 2021-10-07 22:37:30
summary:
categories:
  - 数据库
tags:
  - MySQL
  - 执行计划
---

书接上回 [MySQL的执行计划(二)](https://blog.csdn.net/upstream480/article/details/120615700)

## 执行计划中的列
### possible_keys
显示在查询中使用了哪些索引
![在这里插入图片描述](https://img-blog.csdnimg.cn/75296e9e4b8f420498f3a01f1015d330.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
### key
- 实际使用的索引，如果为NULL，则没有使用索引；
- 查询中如果使用了覆盖索引，则该索引仅出现在key列中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/bbf4082c4e8447c8b4ac51dab855ed62.png)
- possible_keys列表明哪一个索引有助于更高效的查询；
- key是possible_keys的子集；
- 而key列表明实际优化采用了哪一个索引可以更加高效。

### key_len
表示索引中使用的字节数，查询中使用的索的长度（最大可能长度），并非实际使用长度，理论上长度越短越好。
- key_len是根据表定义计算而得的，不是通过表内查询出来的
- 这个字段可以评估组合索引是否完全被使用，这也是我们优化sql时，评估索引的重要指标

### ref
- 表示在key列记录的索引中查找值，所用的列或常量const。

### rows
估算出找到所需行而要读取的行数
![在这里插入图片描述](https://img-blog.csdnimg.cn/f6c9cad466c64e429d039525b18704dc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
这个数字是内嵌循环关联计划里的循环数，它不是最终从表中读取出来的行数，而是MySQL为了找到符合查询的那些行而必须读取行的平均数，只能作为一个相对数来进行衡量。

- 估算该sql返回结果集需要扫描读取的行数，这个值相关重要；
- 索引优化之后，扫描读取的行数越多，说明要么是索引设置不对，要么是字段传入的类型之类的问题，说明要优化空间越大

### filtered
- 返回结果的行数占读取行数的百分比，值越大越好；
- 百分比越高，说明需要查询到数据越准确；
- 百分比越小，说明查询到的数据量大，而结果集很少。

### extra
额外信息

**Using index**
表示SQL中使用了覆盖索引。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e759b9b2122e427f9744249ce088b986.png)

**Using where**
许多where条件里是涉及索引中的列，当它读取索引时，就能被存储引擎检验，因此不是所有带·where子句的查询都会显示“Using where”。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4347858fe7c64a4da610448b50cdcfa8.png)

**Using temporary**
- 对查询结果排序时，使用了一个临时表，常见于order by 和group by
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ba88e162d504c368a7be2cf126e2308.png)


**Using filesort**
- 对数据使用了一个外部的索引排序，而不是按照表内的索引进行排序读取；
- MySQL无法利用索引完成的排序操作成为“文件排序”。
![在这里插入图片描述](https://img-blog.csdnimg.cn/93c3a906083f4366af3623533e092f44.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)


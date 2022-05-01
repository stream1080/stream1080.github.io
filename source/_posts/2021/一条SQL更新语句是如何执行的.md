---
title: 一条SQL更新语句是如何执行的
top: false
cover: false
toc: true
mathjax: false
abbrlink: 35870
date: 2021-08-23 16:44:27
author:
img:
coverImg:
password:
summary:
categories:
  - 数据库
tags:
  - 更新语句
---

## MySQL日志文件
### 二进制日志-binlog
- 二进制日志，其实就是我们平常所说的 binlog，它是 MySQL 重要的日志模块，在 Server 层实现。

- binlog 以二进制形式，将所有修改数据的 query 记录到日志文件中，包括 query 语句、执行时间、相关事务信息等。

### InnoDB-redo log
- redo log，是存储引擎 InnoDB 生成的日志，主要为了保证数据的可靠性
- redo log 记录了 InnoDB 所做的所有物理变更和事务信息。
- redo log 默认存放在数据目录下面，可以通过修改 innodb_log_file_size 和 innodb_log_files_in_group 来配置redo log 的文件数量和每个日志文件的大小。

### 错误日志-error log
- 错误日志，记录 MySQL 每次启动关闭的详细信息，以及运行过程中比较严重的警告和错误信息。
- 错误日志默认是关闭的，可以通过配置参数 log-error 进行开启，以及指定存储路径。

### 慢查询日志-slow query log
- 慢查询日志，记录 MySQL 中执行时间较长的 query，包括执行时间、执行时长、执行用户、主机等信息。
- 慢查询日志默认是关闭的，可以通过配置 slow_query_log 进行开启
- 慢查询的阈值和存储路径，通过配置参数 long_query_time 和 slow_query_log_file 实现。

### 一般查询日志-general query log
- 一般查询日志，记录 MySQL 中所有的 query
- 慢查询记录的是超过阈值的 query，而一般查询日志记录的是所有的 query。
- 一般查询日志的开启需要慎重，因为开启后对 MySQL 的性能有比较大的影响。
- 一般查询日志默认是关闭的，可以通过配置参数 general_log 进行开启
- 存储路径可以通过配置参数 general_log_file 来实现

### binlog和redo log的区别
- binlog是逻辑日志，记录SQL语句的基本逻辑；redo log是物理日志，记录在数据页所做的修改；
- binlog是在Server层实现，所有的存储引擎都可以使用binlog这个日志模块；redo log是InnoDB存储引擎特有的日志模块；
- binlog是追加写，在写满或重启之后，会生成新的binlog文件，之前的日志不会进行覆盖；redo log是循环写，空间大小是固定的；
- binlog 是在事务最终提交前写入的；redo log是在事务执行过程不断的写入；
- binlog可以应用于数据归档、主从搭建等场景；redo log作为异常宕机或者介质故障后的数据恢复使用；

## 一条更新语句是如何执行的
更新语句的执行流程比查询语句多了两个重要的日志模块： binlog（归档日志）和redo log（重做日志）
```
update student set sex='女' where name=' 李四 ';
```
1. 先查询到李四这一条数据，如果命中缓存会之间返回
2. 执行器拿到查询的数据，把 sex 改为 女，然后调用引擎 API 接口，写入这一行数据，
3. InnoDB 引擎把数据保存在内存中，同时记录 redo log，此时 redo log 进入 prepare 状态，
4. 执行器生成更新操作的 binlog，并写入磁盘
5. 调用引擎接口提交事务，更新 redo log为commit状态，更新完成。

上述过程便是 redo log 两阶段提交，如果不这样会发生什么
#### 先redo log后binlog
- 假设更新完 redo log 后，数据库宕机，没有被写入binlog 日志
- 数据库重启后会通过 redo log 恢复数据，但是 bingog 并没有记录这个更新
- 就会丢失这一条数据，主从同步也会丢失这一条数据
#### 先binlog后redo log
- 假设先写了 binlog，数据库宕机，没有更新redo log
- 数据库重启后是无法恢复这一条记录的
- 但是binlog又有记录，就会导致数据不一致
## 总结
分析器--->权限校验--->执行器--->执行引擎--->redo log prepare--->binlog日志--->redo log commit


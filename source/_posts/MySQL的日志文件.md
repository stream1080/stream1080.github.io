---
title: MySQL的日志文件
cover: false
top: false
date: 2021-10-04 20:44:52
summary:
categories:
  - 数据库
tags:
  - MySQL
  - 日志
---

## 二进制日志-binlog
二进制日志-binlog，它是 MySQL 重要的日志模块，在 Server 层实现

binlog 以二进制形式，将所有修改数据的 query 记录到日志文件中，包括：
- query 语句
- 执行时间
- 相关事务信息

binlog 的开启，通过在配置文件 my.cnf 中，显式指定参数 log-bin=file_name
- statement记录的是SQL语句，最后会有COMMIT。
- row记录的实际操作的数据记录，最后会有一个XID event。
### binlog的三种工作模式
#### Row level：
日志中会记录每一行数据被修改的情况，然后在slave端对相同的数据进行修改。
- 优点：能清楚的记录每一行数据修改的细节
- 缺点：数据量太大
#### Statement level（默认）
每一条被修改数据的sql都会记录到master的bin-log中，slave在复制的时候sql进程会解析成和原来master端执行过的相同的sql再次执行。在主从同步中一般是不建议用statement模式的，因为会有些语句不支持，比如语句中包含UUID函数，以及LOAD DATA IN FILE语句等
- 优点：解决了 Row level下的缺点，不需要记录每一行的数据变化，减少bin-log日志量，节约磁盘IO，提高新能
- 缺点：容易出现主从复制不一致
### Mixed（混合模式）
结合了Row level和Statement level的优点，但结构也更复杂。
### binlog的结构
- **timestamp** 事件开始的执行时间，固定4字节展示是新纪元(epoch time)以来的秒数
- **Event** Type 指明该事件的类型
- **server_ id** 服务器的server ID
- **Event size** 该事件的长度
- **Next_ log pos** 固定4字节下一个event的开始位置
- **Flag**   固定2字节event flags
- **Fixed part** 每种Event Type对应结构体固定的结构部分
- **Variable part** 每种Event Type对应结构体可变的结构部分

## 重做日志-redo log
redo log是存储引擎 InnoDB 生成的日志，主要为了保证数据的可靠性。redo log 记录了 InnoDB 所做的所有物理变更和事务信息。
由两部分组成：
- 一是内存中的重做日志缓冲（redo log buffer），其是易失的；
- 二是重做日志文件（redo log file），它是持久的


## binlog和redolog的区别

1. redolog是在InnoDB存储引擎层产生，而binlog是MySQL数据库的上层服务层产生的。
2. 两种日志记录的内容形式不同。MySQL的binlog是逻辑日志，其记录是对应的SQL语句，对应的事务。而innodb存储引擎层面的重做日志是物理日志，是关于每个页（Page）的更改的物理情况。
3. 两种日志与记录写入磁盘的时间点不同，binlog日志只在事务提交完成后进行一次写入。而innodb存储引擎的重做日志在事务进行中不断地被写入，并日志不是随事务提交的顺序进行写入的。
4. binlog不是循环使用，在写满或者重启之后，会生成新的binlog文件，redolog是循环使用。
5. binlog可以作为恢复数据使用，主从复制搭建，redolog作为异常宕机或者介质故障后的数据恢复使用。

## 回滚日志-undo log
重做日志记录了事务的行为，可以很好地通过其对页进行“重做”操作。
但是事务有时还需要进行回滚操作，这时就需要undo。因此在对数据库进行修改时，InnoDB存储引擎不但会产生redo，还会产生一定量的undo。这样如果用户执行的事务或语句由于某种原因失败了，又或者用户用一条ROLLBACK语句请求回滚，就可以利用这些undo信息将数据回滚到修改之前的样子。

redo存放在重做日志文件中，与redo不同，undo存放在数据库内部的一个特殊段（segment）中，这个段称为undo段（undo segment），undo段位于共享表空间内。


## 错误日志-error log
错误日志，记录 MySQL 每次启动关闭的详细信息，以及运行过程中比较严重的警告和错误信息。

## 慢查询日志-slow query log
慢查询日志，记录 MySQL 中执行时间较长的 query，包括执行时间、执行时长、执行用户、主机等信息。

慢查询日志默认是关闭的，可以通过配置 slow_query_log 进行开启。慢查询的阈值和存储路径，通过配置参数 long_query_time 和 slow_query_log_file 实现。

```xml
slow_query_log = 1                              #开启慢查询
long_query_time = 1                             #设置慢查询阈值为1s
slow_query_log_file = /mysql/log/mysql-slow.log #设置慢查询日志存储路径
```


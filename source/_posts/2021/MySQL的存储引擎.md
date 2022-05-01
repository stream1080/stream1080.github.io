---
title: MySQL的存储引擎
top: false
cover: false
toc: true
mathjax: false
abbrlink: 23763
date: 2021-08-24 16:44:14
author:
img:
coverImg:
password:
summary:
categories:
  - 数据库
tags:
  - 存储引擎
  - MySQL
---

# MySQL存储引擎
MySQL 提供不同的技术存储数据，这些技术使用不同的数据存储机制、索引建立方式、锁方式来完成数据的构建，这些技术统称为存储引擎。

MySQL 至少支持 9 种存储引擎，目前最受关注的是 InnoDB 和 MyISAM 存储引擎

##  MyISAM
- MySQL5.5版之前的默认数据库引擎，性能不错，而且提供了大量的特性，包括全文索引、压缩、空间函数等
- 但MyISAM不支持事务和行级锁， 而且崩溃后无法安全恢复。
## InnoDB
- 由于MyISAM的不足，MySQL引入了InnoDB (事务性数据库引擎)
- MySQL 5. 5版本后默认的存储引擎为InnoDB，我们使用的基本都是InnoDB 存储引擎
- 但是在某些情况下使用MyISAM 也是合适的比如读操作非常频繁的情况下。
## 区别

- 是否支持行级锁: MyISAM 只有表级锁, 而InnoDB 支持行级锁和表级锁 ，默认为行级锁。
- 是否支持事务和崩溃后的安全恢复: MyISAM 强调的是性能，每次查询具有原子性，其执行速度比InnoDB更快，但是不提供事务支持。
- InnoDB提供事务支持事务，外部键等高级数据库功能。具有事务、回滚和崩溃修复能力的事务安全表。
- 是否支持外键: MyISAM不支持，而InnoDB支持。
- 是否支持MVCC :InnoDB支持，MyISAM不支持。应对高并发事务，MVCC比单纯的加锁更高效;
- MVCC只在 读取已提交和可重复读两个隔离级别下工作;
- MVCC可以使用乐观锁 和悲观锁来实现

## 使用场景
- MyISAM 对于不支持事务并且存在大量 SELECT 的读场景比较合适；

- 如果业务代码中要支持事务，必须选择 InnoDB 存储引擎；

- 如果业务代码中要支持外键，必须选择 InnoDB 存储引擎；



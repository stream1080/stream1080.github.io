---
title: Redis的持久化方式
top: false
cover: false
toc: true
mathjax: false
categories:
  - 缓存
tags:
  - Redis
  - 持久化
abbrlink: 2050
date: 2021-08-19 17:00:35
author:
img:
coverImg:
password:
summary:
---

# Redis的持久化方式
Redis 缓存的优势是提供快速的查询和存储能力，所以所有的数据都被存储在内存中。相对于硬盘，内存中的数据是半持久化存储，当遇到不可抗阻力，例如断电或者硬件损坏导致的服务器宕机时，内存中的数据会完全丢失。为了防止 Redis 中的数据丢失，需要将数据持久化存储到硬盘。
#### 持久化
持久化就是把 Redis 数据从内存同步到硬盘的过程。Redis中提供了两种持久化方案

- RDB 持久化：将 Redis 的数据定时 dump 到硬盘；
- AOF 持久化：将 Redis 的操作日志追加写入硬盘文件。

## RDB方式
RDB方式也叫快照持久化
- Redis可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。
- Redis创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本(Redis主从结构，主要用来提高Redis性能)
- 还可以将快 照留在原地以便重启服务器的时候使用。

快照持久化是Redis默认采用的持久化方式，在redis.conf配置文件中默认有此下配置
```
save 900 1    
# 在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 300 10
# 在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 60 10000
# 在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

```
#### RDB过程
RDB 持久化方式的步骤是：

- 定时从 Redis 主进程 fork 一个 Redis 子进程；
- Redis 子进程生成内存的数据快照，并且写入 RDB 临时文件；
- 临时文件写入成功后，生成 RDB 最终文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ad270ae5e9504d578e59ad3990fea4ef.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
#### 触发机制
RDB 同步有三种触发机制：save 指令、bgsave 指令、自动化指令
- save 指令：命令会阻塞当前 Redis 服务器，导致 Redis 暂时不可用；
- bgsave 指令：命令在后台异步执行操作，Redis 服务器同时也能响应客户端请求，阻塞只发生在 fork 时间段，时间很短。
- 自动化指令：在 redis.conf 文件配置，例如数据在经过 m 次修改后自动触发 bgsave 命令(上面说的哪种)

简单来说，因为 save 指令会在整个过程中阻塞服务器，所以线上生产环境都是使用 bgsave 指令。

## AOF方式
AOF 持久化方式以日志形式记录 Redis 服务器的每一个插入、修改、删除操作，以文本的形式异步保存。
- AOF 持久化与快照持久化相比，AOF持久化的实时性更好，因此已成为主流的持久化方案。
- 默认情况下Redis没有开启AOF (append only file)方式的持久化
- 可以通过配置文件的appendonly参数开启:appendonly yes

#### AOF过程
开启AOF持久化后每执行条会更改Redi s中的数据的命令 ，Redis就会将该命令写入硬盘中的AOF 文件。
AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是appendonly.aof。
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d34b532eb754841b4253afb583812ee.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
在Redis的配置文件中存在三种不同的AOF持久化方式，它们分别是:
```
appendfsync always     # 每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
appendfsync everysec   # 每秒钟同步一次，显示地将多个写命令同步到硬盘
appendfsync no         # 让操作系统决定何时进行同步
```
- 为了兼顾数据和写入性能，一般采用appendfsync everysec选项 ；
- 让Redis每秒同步一 次AOF文件，Redis性能几乎没受到任何影响；
- 而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据；
- 当硬盘忙于执行写入操作的时候，Redis还会优雅的放慢自己的速度以便适应硬盘的最大写入速度。

## 对比
#### RDB 的优点：
- 性能较高：在开始持久化时，只需要 fork 出一个子进程，子进程负责持久化的工作，避免主线程的 IO 操作；
- 启动效率高：相对于 AOF 方式，如果数据集非常大， 启动速度更快。

#### RDB 的缺点：
- 如果服务器在定时持久化之前宕机，那么 Redis 中没来得及写入的数据都会丢失。

#### AOF 的优点：
- 数据安全：每次修改都会同时追加到日志文件，就算服务器宕机，数据也能在日志文件中找回；
- 方便重建：日志文件同 MySQL 的 binlog 功能相同，我们所有的写操作都能从日志文件中获取。

#### AOF 的缺点：
- 运行效率低：AOF 方式本质上是牺牲缓存的性能，来换去缓存的一致性；
- 占用文件更大：AOF 的日志文件相对 RDB 同步的文件，通常要更大，所以在恢复时速度更慢。

## 混合持久化
- Redis 4.0开始支持RDB和AOF的混合持久化
- 默认关闭，可以通过配置项aof-use-rdb-preamble开启

在混合持久化下，AOF重写的时候就直接把RDB 的内容写到AOF文件开头。这样做的好处是可
以结合RDB和AOF的优点，快速加载同时避免丢失过多的数据。当然缺点也是有的，AOF 里面的
RDB部分是压缩格式不再是AOF格式，可读性较差。

#### AOF重写
- AOF重写可以产生一个新的AOF文件，这个新的AOF文件和原有的AOF文件所保存的数据库状态一样，但体积更小。
- AOF重写是通过读取数据库中的键值对来实现的，程序无须对现有AOF文件进行任何读入、分析或者写入操作。
- 在执行BGREWRITEAOF 命令时，Redis 服务器会维护一个AOF 重写缓冲区，该缓冲区会在子进程创建
新AOF文件期间，记录服务器执行的所有写命令。
- 当子进程完成创建新AOF文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新AOF文件的末尾，使得新旧两个AOF文件所保存的数据库状态一致。
- 最后，服务器用新的A0F文件替换旧的AOF文件，以此来完成AOF文件重写操作



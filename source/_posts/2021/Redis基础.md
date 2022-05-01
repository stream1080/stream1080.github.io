---
title: Redis基础
top: false
cover: false
toc: true
mathjax: false
abbrlink: 26216
date: 2021-03-22 22:45:36
author:
img:
coverImg:
password:
summary:
categories:
  - 缓存
tags:
  - Redis
---

# Redis介绍
- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。


# Redis的数据结构
## Redis有五种数据结构
#### 字符串(String)
- 字符串类型是非常常见的一种类型，Redis中的字符串类型和很多编程语言里的字符串类型差不多，但相对要灵活些。
#### 字符串列表(list)
- 列表可以看作是个双端队列，可以在列表两端推入和弹出元素。对列表的不同操作可以实现其他编程语言中的堆栈（同一端进出）和队列（一端进，另一端出）数据结构。

#### 字符串集合(set)
- set：集合最显著的特点应该就是其中的元素互不相同。用户可以快速对集合进行插入，删除，检查某元素是否在集合中的操作。此外，多个集合间也能很方便的执行交、并、差集运算。

#### 哈希(hash)
- hash：Redis中的散列可以让用户将多个键值对存储到一个Redis键中，可以把这种数据聚集看作是关系数据库中的行，或者文档数据库中的文档。

#### 有序字符串集合(sorted set)
- zset：有序集合存储着成员与分值（权值，在Redis中以IEEE 754双精度浮点数的格式存储）之间的映射，并且提供了分值处理命令。适用于按照权值获取元素的情况。如热门帖子获取、基于投票数排序文章等。



# Redis常用命令

### 字符串(String)
- set key value     		# 设置key对应的值为string类型的value
- get key 						# 获取指定key的value值
- del key   					# 删除指定key的值
- getset key  value 		#先获取key的值，再设置key的值为value
- incr key						#对key的值做自增操作，返回新的值
- decr key					#对key的值做自减操作，返回新的值
- incrby key integer		#对key的value增加指定数值
- decrby key integer	#对key的value减指定数值
- append key value		#给指定key的字符串追加数值



### 哈希(hash)
String Key和String Value的map容器
- hset key field value	向散列key中添加一个键值对
- hget key field	获取散列key中的一个键的值
- hmset key field1 value1 [field2 value2 ...]	向散列key中添加一个或多个键值对
- hmget key field1 [field2 ... ]	获取散列key中的一个或多个键的值
- hdel key field1 [field2 ... ]	删除散列中的一个或多个键值对，返回成功删除的键值对数量
- hlen key	返回散列包含的键值对数量
- hincrby key field integer	给散列中指定field的值加上integer
- hexists key field	测试指定field是否在散列中存在
- hkeys key	返回散列中所有field
- hvals key	返回散列中所有value
- hgetall key	返回散列中所有键值对

### 字符串列表(list)

#### 类似数据结构中的链表，有以下三种存储方式
- ArrayList使用数组方式存储
- LinkedList使用双向链接方式
- 双向链表中增加数据
- 双向链表中删除数据

#### 常用list命令 
- lpush key value1 [value2...]	在key对应的list的头部（左端）添加元素
- rpush key value1 [value2...]	在key对应的list的尾部（右端）添加元素
- lpop key	移除并返回key对应的list的头部（左端）元素
- rpop key	移除并返回key对应的list的尾部（右端）元素
- lindex key offset	返回列表中偏移量为offset的元素
- lrange key start end (0代表第一个元素，-1代表最后一个元素)	返回列表中从start偏移量到end偏移量中的元素，包含两个端点元素（只返回，原list不变）
- ltrim key start end	对列表进行修剪，只保留从start偏移量到end偏移量的元素，包含两个端点
- llen key	返回对应列表的长度

#### 其他命令
- blpop key1 [key2...] timeout	从第一个非空列表中弹出位于头部（左端）的元素，或者在timeout秒内阻塞并等待可弹出元素的出现
- brpop key1 [key2...] timeout	从第一个非空列表中弹出位于尾部（右端）的元素，或者在timeout秒内阻塞并等待可弹出元素的出现
- rpoplpush source-key dest-key	从source-key列表中弹出位于最右端的元素，然后把这个元素插入dest-key列表的最左端，并向用户返回该元素
- brpoplpush source-key dest-key timeout	从source-key列表中弹出位于最右端的元素，然后把这个元素插入dest-key列表的最左端，并向用户返回该元素。如果source-key为空，则在timeout秒内阻塞并等待可弹出元素的出现

### 字符串集合(set)
#### 常用命令
- sadd key value1 [value2...]	将一个或多个元素添加到集合中，并返回添加成功的元素的个数
- srem key value1 [value2...]	从集合中移除一个或多个元素，返回移除成功的元素的个数
- sismember key value	检查元素value是否在集合key中，存在返回1，不存在返回0
- scard key	返回集合包含的元素个数
- smembers key	返回集合包含的全部元素
- srandmember key [count]	随机返回集合中的一个或多个元素。当count为正数时，返回的元素不会重复；当count为负数时，返回的元素有可能会重复
- spop key	随机移除集合中的某个元素，被返回给用户
- smove source-key des-key value	如果集合source-key中包含元素value，则移除它，并添加到dest-key中；成功移除并添加则返回1，否则返回0

#### 其他命令
- sdiff key1 [key2...]	返回所有集合的差集
- sdiffstore dest-key key1 [key2...]	将计算所得的差集中的元素添加到dest-key中
- sinter key1 [key2...]	返回所有集合的交集
- sinterstore dest-key key1 [key2...]	将计算所得的交集中的元素添加到dest-key中
- sunion key1 [key2...]	返回所有集合的并集
- sunionstore dest-key key1 [key2...]	将计算所得的并集中的元素添加到dest-key中


### 有序字符串集合(sorted set)
- zadd key score1 member1 [score2 member2 ... ]	将带有给定分值的成员你添加到有序集合中。注意分值在前，成员在后
- zrem key member1 [member2 ... ]	从有序集合中删除指定成员，并返回成功删除的元素个数
- zcard key	返回有序集合包含的成员个数
- zincrby key increment member	给指定成员的分值加上increment
- zcount key min max	返回分值介于min和max之间的成员数量
- zrank key memeber	返回指定成员在有序集合中的排名，成员按照score从小到大排序
- zrevrank key memeber	返回指定成员在有序集合中的排名 ，成员按照score从大到小排序
- zscore key member	返回指定成员的分值
- zrange key start stop [WITHSCORES]	返回有序集合中排名介于start和stop之间的成员，如果给定了可选的 WITHSCORES选项，则连带成员分值一并返回
- zrevrange key start stop [WITHSCORES]	同上，结果按score逆序
- zrangebyscore key min max [WITHSCORES]	返回分值介于min和max之间的成员
- zremrangebyrank key start stop	删除集合中排名在start和stop之间的成员
- zremrangebyscore key min max	删除集合中分值在min和max之间的成员


### key的相关命令
#### Key定义注意点
- 不要过长
- 不要过短
- 在项目中应该遵循统一的命名规范

#### 常用命令
- exists key	测试指定key是否存在
- del key1 [key2 ...]	删除给定key
- type key	返回给定key的类型
- keys pattern	以正则表达式的形式，返回匹配的所有key 
- keys *(返回所有key)
- rename oldkey newkey	修改key的名字
- dbsize	查看数据库中key的数量（并不是所谓的大小）
- expire key seconds	为key指定过期时间，到期自动删除
- ttl key	查看key的剩余时间
- select index	选择数据库（默认16个数据库，index为0-15）
- move key index	将key移动到指定数据库
- flushdb	清空当前数据库key
- flushall	清空所有数据库key

# Redis的事务
#### Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

#### 一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

## 事务的常用命令

- discard 取消事务，放弃执行事务块内的所有命令。
- exec 执行所有事务块内的命令。
- multi 标记一个事务块的开始。
- umwatch 取消 WATCH 命令对所有 key 的监视。
- watch key [key ...]
监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

# Redis的持久化-数据从内存同步到硬盘

## 两种持久化方式：
- RDB方式
- AOF方式

## 持久化使用的方式：
- RDB方式：
默认支持，在指定的时间间隔内，将内存中的数据集快照写入到磁盘
- AOF方式：
日志的形式记录服务器处理的每一个操作，服务器启动之初，读取文件，重新构建数据库
- 无持久化：
通过配置继用Redis持久化功能，Redis缓存机制
- 同时使用RDB和AOF

## RDB方式的持久化
#### 优势：
- 数据库只包含一个文件，通过文件备份策略，定期配置，恢复系统灾难
- 压缩文件转移到其他介质上
- 性能最大化，redis开始持久化时，分叉出进程，由子进程完成持久化的工作
，避免服务器进程执行I/O操作，启动效率高

#### 劣势：
- 无法高可用：系统一定在定时持久化之前宕机，数据还没写入，数据已经丢失
- 通过fock分叉子进程完成工作，数据集大的时候，服务器需要停止几百毫秒甚至1秒

#### 默认配置配置：查看redis.conf配置文件
- save 900 1 #每900秒至少1个key变化，持久化一次，到内存一个快照
- save 300 10 #每300秒至少10个key变化，往硬盘写一次
- save 60 10000 #每60秒至少10000个key变化，写一次
- dbfilename dump.rdb #数据的文件名
- dir ./ #保存的路径，redis路径下

## AOF持久化方式

#### 优点：
- 同步写入频率高
- 不破坏写入日志数据
- 当数据过大，可启动修改重写机制，保证修改数据的更新
- 日志文件格式清晰，便于重建数据

#### 缺点：
- 效率低
- 文件偏大于rdb文件

#### 配置过程：
- 编辑redis.conf配置文件
- 找到 appendonly no 修改为 yes

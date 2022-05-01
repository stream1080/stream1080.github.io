---
title: Java集合之HashMap(二)
top: false
cover: false
toc: true
mathjax: false
abbrlink: 32182
date: 2021-07-22 20:40:59
author:
img:
coverImg:
password:
summary:
categories:
  - Java基础
tags:
  - HashMap
---

## HashMap扩容机制
### 明确几个参数：
- capacity 即容量，默认16。
- loadFactor 加载因子，默认是0.75
- threshold 阈值。阈值=容量*加载因子。默认12。当元素数量超过阈值时便会触发扩容。

### 什么时候触发扩容？
- 一般情况下，当元素数量超过阈值时便会触发扩容。每次扩容的容量都是之前容量的2倍。
- HashMap的容量是有上限的，必须小于1<<30，即1073741824。
- 如果容量超出了这个数，则不再增长，且阈值会被设置为Integer.MAX_VALUE
### 什么是扩容
- 扩容(resize)就是重新计算HashMap的容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。
- 当然Java里的数组是无法自动扩容的，因为数组空间是连续分配的，必须使用一个新的数组代替已有的容量小的数组，就像我们用一个小瓶子装水，水装满之后如果想装更多的水，就得换大瓶子。

### HashMap扩容源码
#### JDK1.7
```java
    Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
    transfer(newTable);                         //！！将数据转移到新的Entry数组里
    table = newTable;                           //HashMap的table属性引用新的Entry数组
    threshold = (int)(newCapacity * loadFactor);//修改阈值
```
- 这里就是使用一个容量更大的数组来代替已有的容量小的数组
- transfer()方法将原有Entry数组的元素拷贝到新的Entry数组里
#### JDK1.8
```java
 1 final Node<K,V>[] resize() {
 2     Node<K,V>[] oldTab = table;
 3     int oldCap = (oldTab == null) ? 0 : oldTab.length;
 4     int oldThr = threshold;
 5     int newCap, newThr = 0;
 6     if (oldCap > 0) {
 7         // 超过最大值就不再扩充了
 8         if (oldCap >= MAXIMUM_CAPACITY) {
 9             threshold = Integer.MAX_VALUE;
10             return oldTab;
11         }
12         // 没超过最大值，就扩充为原来的2倍
13         else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
14                  oldCap >= DEFAULT_INITIAL_CAPACITY)
15             newThr = oldThr << 1; // double threshold
16     }
17     else if (oldThr > 0) // initial capacity was placed in threshold
18         newCap = oldThr;
19     else {               // zero initial threshold signifies using defaults
20         newCap = DEFAULT_INITIAL_CAPACITY;
21         newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
22     }
23     // 计算新的resize上限
24     if (newThr == 0) {
25 
26         float ft = (float)newCap * loadFactor;
27         newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
28                   (int)ft : Integer.MAX_VALUE);
29     }
30     threshold = newThr;
31     @SuppressWarnings({"rawtypes"，"unchecked"})
32         Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
33     table = newTab;
34     if (oldTab != null) {
35         // 把每个bucket都移动到新的buckets中
36         for (int j = 0; j < oldCap; ++j) {
37             Node<K,V> e;
38             if ((e = oldTab[j]) != null) {
39                 oldTab[j] = null;
40                 if (e.next == null)
41                     newTab[e.hash & (newCap - 1)] = e;
42                 else if (e instanceof TreeNode)
43                     ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
44                 else { // 链表优化重hash的代码块
45                     Node<K,V> loHead = null, loTail = null;
46                     Node<K,V> hiHead = null, hiTail = null;
47                     Node<K,V> next;
48                     do {
49                         next = e.next;
50                         // 原索引
51                         if ((e.hash & oldCap) == 0) {
52                             if (loTail == null)
53                                 loHead = e;
54                             else
55                                 loTail.next = e;
56                             loTail = e;
57                         }
58                         // 原索引+oldCap
59                         else {
60                             if (hiTail == null)
61                                 hiHead = e;
62                             else
63                                 hiTail.next = e;
64                             hiTail = e;
65                         }
66                     } while ((e = next) != null);
67                     // 原索引放到bucket里
68                     if (loTail != null) {
69                         loTail.next = null;
70                         newTab[j] = loHead;
71                     }
72                     // 原索引+oldCap放到bucket里
73                     if (hiTail != null) {
74                         hiTail.next = null;
75                         newTab[j + oldCap] = hiHead;
76                     }
77                 }
78             }
79         }
80     }
81     return newTab;
82 }
```
### HashMap扩容过程
扩容的时候，使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
![在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/96e3364cd31de8c7acd58eba62d739df.png)
- 第一个是扩容前的key1和key2两种key确定索引位置的示例，
- 第二个是表示扩容后key1和key2两种key确定索引位置的示例，
- 其中hash1是key1对应的哈希与高位运算结果。
- 元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1

因此新的索引就会发生这样的变化：
![在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/0f244ba2da9810f8ed091f94cd418e8c.png)

因此，在扩充HashMap的时候，只需要看看原来的hash值新增的那个bit是1还是0，
是0的话索引没变，是1的话索引变成“原索引+oldCap”


- 这样省去了重新计算hash值的时间
- 而且新增的1位是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。
### 小结
- 扩容是一个特别耗性能的操作，所以当程序员在使用HashMap的时候，估算map的大小，初始化的时候给一个大致的数值，避免map进行频繁的扩容。
- 装载因子是可以修改的，也可以大于1，但是建议不要轻易修改，0.75是一个比较好的折中数字
- HashMap是线程不安全的，不要在并发的环境中同时操作HashMap



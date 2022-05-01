---
title: HashMap的扩容
cover: false
top: false
categories:
  - Java基础
tags:
  - HashMap
  - 扩容
abbrlink: 61579
date: 2021-09-12 21:13:19
summary:
---

## HashMap初始化
在JDK1.8中，定义了HashMap的初始化过程，我们看看他的源码是如果定义这个初始化过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef83dacd0c114c8b92ce95ee8fb5dc8f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
可以看到，它的构造方法中传入了两个参数，一个是初始化容量，一个是加载因子，默认是0.75f

```java
HashMap(int initialCapacity, float loadFactor)
```
但是这个容量最终调用了另一个构造方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ba346c1aaa74ba6bb213d36903d2ba0.png)
这个threshold的成员变量，就是触发HashMap扩容的阈值，当HashMap的数据量达到或超过threshold时，就会扩容。

我们再往下看这个tableSizeFor(int cap)的构造方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c1e459e5575442c97e23fb351f9d51c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
在 tableSizeFor(int cap)这个构造方法中，对传入的参数cap进行了多次位运算，这样可以让返回值保持在 2 的 N 次方，在扩容的时候，可以快速计算数据在扩容后的新表中的位置。

## HashMap 的 table 初始化
从上面的源码大家可以发现一个问题，整个计算阈值的过程中，装载因子loadFactor并没有参与运算

实际上，在HashMap中，所有的数据都是存储在数组中，这个数组的大小就是阈值与加载因子的乘积

```java
table.size == threshold * loadFactor
```

## HashMap的动态扩容
- 在 HashMap 中，动态扩容是 resize() 方法
- 这个方法了 table 的扩容，它还承担了 table 的初始化。

我们先看看HashMap中put一个元素的过程，最终调用的是putVal这个方法，我们看看源码

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
在resize()中，它调整了扩容阈值threshold，并且完成了对table的初始化。我们看看resize()的源码

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
        	//当我们指定了初始容量，且 table 未被初始化时，oldThr 就不为 0，
        	//将 newCap 赋值为 oldThr，新创建的 table 会是我们构造的 HashMap 时指定的容量值。
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
        	//通过装载因子（loadFactor）调整了新的阈值（newThr）
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 使用 loadFactor 调整后的阈值，重新保存到 threshold 中
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //通过 newCap 创建新的数组，将其指定到 table 上，完成 table 的初始化
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

```

- 所有我们传递进来的 initialCapacity 虽然经过 tableSizeFor() 方法调整后，直接赋值给 threshold
- 但是它实际是 table 的大小，并且最终会通过 loadFactor 重新调整 threshold。

## 经典面试题
### 计划用HashMap存1k条数据，构造时传1000会触发扩容吗
-  HashMap 初始容量指定为 1000，会被 tableSizeFor() 调整为 1024；
- 但是它只是表示 table 数组为 1024；
- 负载因子是0.75，扩容阈值会在 resize() 中调整为 768（1024 * 0.75）
- 会触发扩容

如果需要存储1k的数据，应该传入1000 / 0.75(1333)
- tableSizeFor() 方法调整到 2048，不会触发扩容。

### 计划用HashMap存1w条数据，构造时传10000会触发扩容吗
- 当我们构造HashMap时，参数传入进来 1w 
- 经过 tableSizeFor() 方法处理之后，就会变成 2 的 14 次幂 16384
- 负载因子是 0.75f，可存储的数据容量是 12288（16384 * 0.75f）
- 完全够用，不会触发扩容

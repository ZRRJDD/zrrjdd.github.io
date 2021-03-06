# 什么是ConcurrentHashMap



## 读取本文首先带着下面问题来读：
- 1.在并发场景下，ConcurrentHashMap是怎么保证线程安全的？又是怎么实现高性能都写的？



HashMap不是线程安全的。在高并发环境下做插入操作，有可能出现下面的环形链表：

![环形链表](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492887_20200129223133648_288513406.png)

想要避免HashMap的线程安全问题有很多方法，比如改用HashTable或者Collections.synchronizedMap。**但是，这两者有着共同的问题：性能。无论读操作还是写操作，它们都会给整个集合加锁，导致同一时间的其他操作为之阻塞。**

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492887_20200129223712891_26739716.png)
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492888_20200129223725446_590324589.png)

在并发环境下，如何能够兼顾线程安全和运行效率呢？这时候ConcurrentHashMap就应运而生了。

### Q：比起HashMap，ConcurrentHashMap有什么特别之处呢？

掌握HashMap之后，学习ConcurrentHashMap其实很简单，最关键是理解一个概念：**[Segment]**

**Segment是什么呢？Segment本身就相当于一个HashMap对象。**

同HashMap一样，Segment包含一个HashEntry数组，数组中的每一个HashEntry既是一个键值对，也是一个链表的头节点。

单一的Segment结构如下：

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492888_20200129224207762_210918183.png)

像这样的Segment对象，在ConcurrentHashMap集合中有多少个呢？有2的N次方个，共同保存在一个名为**segments**的数组当中。

因此，整个ConcurrentHashMap的结构如下：

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492889_20200129224411491_1553884131.png)


**可以说，ConcurrentHashMap是一个二级哈希表。在一个总的哈希表下面，有若干个子哈希表。** 这样的二级结构，和数据库的水平拆分有些相似。

### Q：ConcurrentHashMap这样设计 有什么好处呢 ？

ConcurrentHashMap优势就是采用了**[锁分段技术]**，每一个Segment就好比一个自治区，读写操作高度自治，Segment之间互不影响。

下面我们来看看ConcurrentHashMap并发读写的几种情形：
#### Case1：不同Segment的并发写入
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492890_20200129225023211_581958550.png)

不同Segment的写入时可以并发执行的。

#### Case2：同一Segment的一写一读
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492890_20200129225125590_1336821412.png)

同一Segment的写和读是可以并发执行的。

#### Case3：同一Segment的并发写入
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492891_20200129225926581_983556570.png)

Segment的写入时需要上锁的，因此对同一Segment的并发写入会被阻塞。

由此可见，**ConcurrentHashMap当中的每个Segment各自持有一把锁。在保证线程安全的同时，降低了锁的粒度，让并发操作效率更高。**

### Q：ConcurrentHashMap的读写过程具体是什么样子呢？

来看一下ConcurrentHashMap的读写的详细过程：

#### Get方法：
- 1.为输入的Key做Hash运算，得到hash值
- 2.通过hash值，定位到对应的Segment对象
- 3.再次通过hash值，定位到Segment当中数组的具体位置。

#### Put方法:
- 1.为输入的Key做Hash运算，得到hash值。
- 2.通过hash值，定位到对应的Segment对象
- 3.获取可重入锁
- 4.再次通过hash值，定位到Segment当中数组的具体位置。
- 5.插入或覆盖HashEntry对象
- 6.释放锁

从步骤可以看出，ConcurrentHashMap在读写时都需要二次定位。首先定位到Segment，之后定位到Segment内的具体数组下标。

### Q：最后一个问题：既然每一个Segment都各自加锁，那么在调用Size方法的时候，怎么解决一致性的问题呢 ？

Size方法的目的是统计ConcurrentHashMap的总元素数量，自然需要把各个Segment内部的元素数量汇总起来。

但是，如果在统计Segment元素数量的过程中，已统计过的Segment瞬间插入新的元素，这时候该怎么办呢？

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492891_20200130131159574_1296041707.png)
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492892_20200130131230062_646338028.png)
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492892_20200130142859332_1030950070.png)
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492892_20200130142927197_1279762494.png)

ConcurrentHashMap的Size方法 是一个嵌套循环，大体逻辑如下：

- 1.遍历所有的Segment
- 2.把Segment的元素数量累加起来。
- 3.Segment的修改次数累加起来。
- 4.判断所有Segment的总修改次数是否大于上一次总修改次数。如果大于，说明统计过程中有修改，重新统计，尝试次数+1；如果不是。说明没有修改，统计结束。
- 5.如果尝试次数通过阈值，则对每一个Segment加锁，再重新统计。
- 6.再次判断所有Segment的总修改次数是否大于上一次的总修改次数。由于已经加锁，次数一定和上次相等。
- 7.释放锁，统计结束。

官方源代码如下：
```java
public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table ,resort to locking
    // 试着数数几次以获得准确的计数。由于表中的连续异步更改而失败时，使用锁定
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum; // sum of modCounts
    long last = 0L; // previous sum
    int retries = -1; //first iteration isnt't retry 第一次迭代不需要审查
    try {
        for (;;){
            if (retires++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0;j < segments.length;++j) {
                    ensureSegment(j).lock(); // force creation  强制创建锁
                }
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0;j < segments.length;++j) {
                Segment<K,V> seg = segmentAt(segments,j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size +=c) <0)
                        overflow = true;
                }
            }
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0;j < segments.length; ++j)
                segmentAt(segments,j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE :size;
}
```

为什么这样设计呢？这种思想和乐观锁悲观锁的思想如出一辙。

为了尽量不锁住所有的Segment，首先乐观地假设Size过程中不会有修改。当尝试一定次数，才无奈转为悲观锁，锁住所有Segment保证强一致性。

## 几点说明

- 1.这里介绍的ConcurrentHashMap原理和代码 ，都是基于Java1.7的。在Java8中会有些许差别。
- 2.ConcurrentHashMap 在对Key求Hash值得时候，为了实现Segment均匀分布，进行了两次Hash。



# 参考链接

- [漫画：什么是ConcurrentHashMap？](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653192083&idx=1&sn=5c4becd5724dd72ad489b9ed466329f5&chksm=8c990d49bbee845f69345e4121888ec967df27988bc66afd984a25331d2f6464a61dc0335a54&scene=21#wechat_redirect)


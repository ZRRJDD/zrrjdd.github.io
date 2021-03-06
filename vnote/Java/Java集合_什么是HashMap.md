# 什么是HashMap



## Q1：说一说HashMap的底层原理？

众所周知，HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做**Entry**。这些键值对（Entry）分散存储在一个数组中，这个数据就是HashMap的主干。

HashMap数组每一个元素的初始值都是Null。

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492756_20200129162154707_1907343449.png)

对于HashMap，我们最常使用的是两个方法：Get 和Put。

### 1.Put方法的原理

调用Put方法的时候发生了什么了呢？

比如调用hashMap.put("apple",0),插入一个key为“apple”的元素。这时候我们需要利用一个哈希函数来确定这个Entry的插入位置（index）：

**index = Hash("apple")**

假定最后计算出的index 是2，那么结果如下：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492756_20200129165208912_1232948574.png)

但是，因为HashMap的长度是有限的，当插入的Entry越来越多时，再完美的Hash函数也难免会出现index冲突的情况。比如下面这样：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492757_20200129165316965_1096973315.png)

这时候该怎么办？我们可以利用**链表**来解决。

HashMap数组的每一个元素不止是一个Entry对象，也是一个链表的头节点。每一个Entry对象通过Next指针指向它的下一个Entry节点。当新来的Entry映射到冲突的数据位置时，只需要插入到对应的链表即可：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492757_20200129165543131_683395710.png)


需要注意的时，新来的Entry节点插入链表时，使用的时“**头插法**”。至于为什么不插入到链表的尾部，后面会有解释。

### 2.Get方法的原理

使用Get方法根据Key来查找Value的时候，发生了什么呢？

首先会把输入的Key做一次Hash映射，得到对应的index：

**index = Hash("apple")**

由于刚才所说的Hash冲突，同一个位置有可能匹配到多个Entry，这时候就需要顺着对应链表的头节点，一个个向下来查找。假定我们要查找的Key是“apple”
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492758_20200129170041033_1089153814.png)



- 第一步，我们查看的是头节点Entry6，Entry6的Key是banana，显然不是我们要找的结果。
- 第二步，我们查看的是Next节点Entry1，Entry1的key是apple，正是我们要找的结果。
- 之所以把Entry6放在头节点，是因为HashMap的发明者认为，**后面插入的Entry被查找的可能性更大**


## Q2：HashMap默认的初始长度是多少？为什么这么规定？
首先明确一点，HashMap的默认初始化长度是16，并且每次自动扩展或者手动初始化时，长度必须是2的幂。

### Q2.1 为什么是16？有什么特殊的意义吗？

之所以选择16，是为了服务于从Key映射到index的Hash算法。之前说过，从Key映射到HashMap数组的对应位置，会用到一个Hash函数。

**index = Hash("apple")**

如何实现一个尽量均匀分布的Hash函数呢？我们通过利用Key的HashCode值来做某种运算。
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492758_20200129171815809_1292157224.png)

**index =  HashCode（Key） % Length ?**

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492759_20200129171850287_1931825304.png)

如何进行位运算呢？有如下的公式（Length是HashMap的长度）：

**Index = HashCode（Key） & （Length - 1)**

下面我们以值为“book”的Key来演示整个过程：
- 1.计算book的hashcode，结果为十进制的3029737，二进制的101110001110101110 1001。
- 2.假定HashMap长度是默认16，计算Length-1的结果为十进制的15，二进制的1111.
- 3.把以上两个结果做**与运算**，101110001110101110 1001 & 1111 = 1001，十进制是9，所以index=9.

可以说，Hash算法最终得到的index结果，完全取决于Key的Hashcode 值得最后几位。
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492759_20200129172704217_382625335.png)
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492760_20200129172716028_1217966227.png)

假设HashMap的长度是10，重复刚才的运算步骤：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492760_20200129172801777_544253985.png)

单独看这个结果，表面上并没有问题。我们再来尝试一个新的HashCode 101110001110101110 1011 ：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492761_20200129173146839_1143873322.png)

让我们再换一个HashCode 101110001110101110 1111 试试：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492761_20200129173230740_805314910.png)

是的，虽然HashCode的倒数第二第三位从0变成了1，但是运算的结果都是1001。也就是说，当HashMap 长度为10的时候，有些index结果的出现几率会更大，而这些index结果永远不会出现（比如0111）！

这样，显然不符合Hash算法均匀分布的原则。

反观长度16或者其他2的幂，Length-1的值是所有二进制位全为1，这种情况下，index结果等同于HashCode后几位的值。只要输入的HashCode本身分布均匀，Hash算法的结果就是均匀的。

## Q3：高并发情况下，为什么HashMap可能会出现死锁？

在分析高并发场景之前，我们需要先搞清楚**Rehash** 这个概念。**Rehash**是HashMap在扩容时候的一个步骤。

HashMap的容量是有限的。当经过多次元素插入，使得HashMap达到一定饱和度时，Key映射位置发生冲突的机率会逐渐提高。

这时候，HashMap 需要扩展它的长度，也就是进行**Resize**
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492762_20200129174522345_1485748976.png)

影响发生**Resize**的因素有两个：
- **1.Capacity**
HashMap的当前长度。上一期曾经说过，HashMap的长度是2的幂。
- **2.LoadFactor**
HashMap负载因子，默认值为0.75f

衡量HashMap是否进行**Resize**的条件如下：

**HashMap.Size >= Capacity * LoadFacotr**

HashMap 的**Resize**不是简单的把长度扩大，而是经过下面两个步骤：
- 1.扩容
创建一个新的Entry空数组，长度是原数组的2倍。
- 2.ReHash
遍历原Entry数组，把所有的Entry重新Hash到新数组。为什么要重新Hash呢？因为长度扩大以后，Hash的规则也随之改变。

让我们回顾一下Hash公式：**index = HashCode（Key）& （Length-1）**

当原数组长豆为8时，Hash运算是和111B做与运算；新数组长度为16，Hash运算是和1111B做与运算。Hash结果显然不同。

Resize前的HashMap：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492762_20200129181153935_43192316.png)

Resize后的HashMap：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492763_20200129181220597_83782326.png)

ReHash的Java代码如下：
```java
/**
 * Transfers all entries from current table to newTable.
 */
void transfer(Entry[] newTable,boolean rehash){
    int newCapacity = newTable.length;
    for (Entry<K,V> e :table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0:hash(e.key);
            }
            int i = indexFor(e.hash,newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

注意：**刚才的流程在单线程执行并没有问题，可惜HashMap并非线程安全的。**
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492763_20200129182951351_1527764297.png)

### 注意：下面的内容十分烧脑，请小伙伴们坐稳扶好。

假设一个HashMap已经到了Resize的临界点。此时有两个线程A和线程B，在同一时刻对HashMap进行Put操作：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492764_20200129183153746_1820056147.png)

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492764_20200129183329461_926594761.png)


此时达到Resize条件，两个线程各自进行Resize的第一步，也就是扩容：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492764_20200129183434878_1874223730.png)

这时候，两个线程都走到了ReHash的步骤与。让我们回顾一下ReHash的代码：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492765_20200129183721572_788095914.png)

假如此时线程B遍历到Entry3对象，刚执行完红框里的这行代码，线程就被挂起。对于线程B来说：
```java
e = Entry3;
next = Entry2;
```
这时候线程A畅通无阻的进行着Rehash，当ReHash完成后，结果如下（图中的额和next，代表线程B的引用）：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492765_20200129184107097_1996375549.png)

直到这一步，看起来没什么毛病。接下来线程B恢复，继续执行属于它自己的ReHash。线程B刚才的状态是：
```java
e = Entry3;
next = Entry2;
```
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492766_20200129184335897_1748770373.png)

当执行到上面这一行时，显然 i = 3,因为刚才线程A对于Entry3的hash结果也是3。
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492766_20200129184443373_1122466169.png)

我们继续执行到这两行，Entry3放入了线程B的数组下标为3的位置，并且**e指向了Entry2**。此时e和next的指向如下：
```java
e = Entry2;
next = Entry2;
```
整体情况如图所示：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492767_20200129184911140_1792547135.png)

接着是新一轮循环，又执行到红框内的代码行：

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492767_20200129195236639_291614182.png)

```java
e = Entry3;
next = Entry3.next = null;
```
最后一步，当我们执行下面这一行的时候，见证奇迹的时刻来了：

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492768_20200129202038472_601874420.png)

```java
newTable[i] = Entry2;
e = Entry3;
Entry2.next = Entry3;
Entry3.next = Entry2;
```
**链表出现了环形**

整体情况如图所示：

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492768_20200129202220579_1055399570.png)

此时，问题还没有直接产生。当调用Get方法查找一个不存在的Key，而这个Key的Hash结果恰好等于3的时候，由于位置3带有环形链表，所以程序将会进入**死循环**

这种情况，不禁让人联想一道经典的面试题：
[漫画算法：如何判断链表有环？](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653189798&idx=1&sn=c35c259d0a4a26a2ee6205ad90d0b2e1&chksm=8c99047cbbee8d6a452fbb171133551553a825c83fb8b0cc66210dcda842c61157a07baaeb6b&scene=21#wechat_redirect)

### 总结：
- 1.HashMap在插入元素过多的时候需要进行Resize，Resize的条件是：**HashMap.Size >= Capacity * LoadFactor **
- 2.HashMap的Resize包含扩容和ReHash两个步骤，ReHash在并发的情况下可能会形成链表环。

## Q4：在Java8当中，HashMap的结构有什么样的优化？




# 参考链接

- [漫画：什么是HashMap？](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653191907&idx=1&sn=876860c5a9a6710ead5dd8de37403ffc&chksm=8c990c39bbee852f71c9dfc587fd70d10b0eab1cca17123c0a68bf1e16d46d71717712b91509&scene=21#wechat_redirect)
- [漫画：高并发下的HashMap](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653192000&idx=1&sn=118cee6d1c67e7b8e4f762af3e61643e&chksm=8c990d9abbee848c739aeaf25893ae4382eca90642f65fc9b8eb76d58d6e7adebe65da03f80d&scene=21#wechat_redirect)





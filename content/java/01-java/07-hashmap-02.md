> 负载因子为什么是0.75？为什么红黑树扩容是当链表长度>=8时扩容？Java7的HashMap扩容死锁演示与环链形成分析？

## 1、负载因子为什么是0.75？

在构造函数中未指定时使用的加载因子

```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

我们知道，最理想的情况就是，当我们 put 进来的元素刚好平铺在数组上，而不产生链表，尽量不产生 Hash 碰撞。但是我们明白这种情况只是理想化的，是难以实现的。

那么我们反证一下：当我们将负载因子不定为0.75的时候（两种情况）：

1. 假如负载因子定为1（最大值），那么只有当元素填慢数组长度的时候才会选择去扩容，虽然负载因子定为1可以最大程度的提高空间的利用率，但是我们的查询效率会变得低下（因为链表查询比较慢）

	**结论：所以当加载因子比较大的时候：节省空间资源，耗费时间资源**

2. 加入负载因子定为0.5（一个比较小的值），也就是说，知道达到数组空间的一半的时候就会去扩容。虽然说负载因子比较小可以最大可能的降低 Hash 冲突，链表的长度也会越少，但是空间浪费会比较大

	**结论：所以当加载因子比较小的时候：节省空间资源，耗费时间资源**

但是我们设计程序的时候肯定是会在空间以及时间上做平衡，那么我们能就需要在时间复杂度和空间复杂度上做折中，选择最合适的负载因子以保证最优化。所以就选择了0.75这个值，JDK 那帮工程师一定是做了大量的测试，得出的这个值吧

## 2、为什么红黑树扩容是当链表长度>=8时扩容

当链表数量大于8时转化为树

![image-20210406194259258](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406194259258.png)

**泊松分布**

当我们在 put 一个元素，产生 hash 冲突的时候，会遵循泊松分布（通过概率学统计出来）的规则；

泊松分布公式：`(exp(-0.5) * pow(0.5, k))`

泊松分布图：

![image-20210406194737078](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406194737078.png)

意思就是说，当负载因子为0.75的时候，还有当链表的长度的增加。如果再添加新的节点进链表的时候，这个添加进当前链表概率是随着节点的增加而越来越少。JDK 源码中有这样的解释：

![image-20210406194933102](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406194933102.png)

也就是说，当 put 进来一个元素，通过 hash 算法，然后最后定为到同一个桶（链表）的概率会随着链表的长度的增加而减少，当这个链表长度为8的时候，这个概率几乎接近于0，所以我们才会将链表转红黑树的临界值定为8。

当然，虽然在 HashMap 底层有这种红黑树的结构，但是我们要知道能产生这种结构的概率也不大，所以我们知道在 JDK1.7 到 JDK1.8 这其中 HashMap 的性能只提高了 7%-8% 左右，提高的并不多。

## 3、Java7 的 HashMap 扩容死锁演示与环链形成分析

JDK1.7 中 put 方法的源码

![image-20210406195416720](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406195416720.png)

我们打开 addEntry 方法如下，它会判断数组当前容量是否已经超过的阈值，例如假设当前的数组容量是16，加载因子为0.75，即超过了12，并且刚好要插入的索引处有元素，这时候就需要进行扩容操作，可以淡道 resize 扩容大小是原数组的两倍，仍然符合数组的长度是2的指数次幂

![image-20210406195658632](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406195658632.png)

我们再进入 resize 方法如下，它首先会对之前的数组容量进行判断，看是否已经达到了数组最大容量，如果没有，后面会进行数组的转移操作，即 transfer 方法

![image-20210406195820564](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406195820564.png)

JDK1.7 中，当数组容量达到 16 * 0.75 = 12 的时候，数组就需要扩容，在扩容的时候，我们都知道会涉及元素的迁移，那么下面代码就是元素迁移的主要代码：JDK1.7 中 HashMap 存在死锁问题的原因也主要集中在这

![image-20210406200044969](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406200044969.png)

我们将重新计算 Hash 值的逻辑先去掉（回顾：为什么使用位运算而不使用取模运算：因为位运算效率高，取模运算效率低，在数组迁移的时候会进行重 Hash，如果效率不高极大的影响 put 的效率）。

只留下主要的迁移数组的代码：

![image-20210406200442988](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406200442988.png)

**举例：**

假设有两个线程，两个线程都对这个 HashMap 进行扩容，扩容就要执行数组迁移代码。

假设线程 T2 先来对数组进行迁移，当线程而二执行到 `Entry<K, V> next = e.next`的时候，然后线程 T2 突然阻塞了；

![image-20210406200726754](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406200726754.png)

外层 for 循环来遍历数组的，当数组的值不是空的时候就进入内层的 while 循环去遍历链表，假设线程 T2 两个指针（一个叫做 e2，一个叫做 next2），当遍历到这个节点的时候：e2 表示的就是大狂神这个节点，next2 表示 e2 的下一个节点，也就是佩雨的这个节点；

![image-20210406201000828](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406201000828.png)

然后刚好线程 T1 也跑进去这个逻辑里面了，那么线程 T1 也有两个指针（e，next），e 也指向的是大狂神，next 也指向的是佩雨

![image-20210406201146087](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406201146087.png)

T1 线程继续跑这个逻辑（下面是 T1 线程自己的扩容与迁移图），会继续转移这个链表，转移完成之后发现一个问题：转移过去后两个节点的上下位置反了（大狂神和佩雨已经反了）

![image-20210406201342551](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406201342551.png)

然后 T2 这个时候又唤醒了，因为线程 T2 一直都是指向这两个元素的（e2 -> 大狂神，next -> 佩雨），也就是成下图这样

![image-20210406201637367](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406201637367.png)

然后 T2 也要去循环执行这个逻辑

![image-20210406201716482](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406201716482.png)

**最后会产生这样的情况，就是最终会形成一个环**

![image-20210406201901067](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406201901067.png)

## 4、Java8 中的 HashMap 扩容优化（不会出现死环）

JDK1.8 中 HashMap 扩容的代码

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
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
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

当我们对数组进行迁移的时候，这里面定义了两组指针，分别是高位和低位的头和尾

```java
HashMap.Node<K, V> loHead = null, loTail = null;
HashMap.Node<K, V> hiHead = null, hiTail = null;
```

然后用当前节点的 hash 值和旧数组的长度（16）做 “与” 运算

```java
if ((e.hash & oldCap) == 0) {
  if (loTail = null)
    loHead = e;
  else
    loTail.next = e;
  loTail = e;
}
```

分析：

![image-20210406203008814](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406203008814.png)

用当前节点的 hash 值和旧数组的长度（16）做 “与” 运算，结果出来如果是0就是低位指针，如果是16就是高位指针，那么我们再看：

```java
// 如果是0，则使用低位的指针
if ((e.hash & oldCap) == 0) {
  if (loTail = null)
    loHead = e;
  else
    loTail.next = e;
  loTail = e;
} else { // 如果是16，则使用高位的指针
  if (hiTail == null)
    hiHead = e;
  else
    hiTail.next = e;
  hiTail = e;
}
```

如果说 “与” 出来的值是0，那么就会用低位的指针去迁移该数组。如果判断出来的是16，就会用高位的指针去迁移。

那么我们的最终的指针就会成下面的样子：

![image-20210406203549470](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406203549470.png)

然后就开始迁移

**低位迁移：首先：先将高位和低位断开（将低位的尾部的 next 置为空）**

```java
if (loTail != null) {
  // 把低位的尾部节点的 next 值为空（先将高位和低位断开）
  loTail.next = null;
  newTab[j] = loHead;
}
if (hiTail != null) {
  hiTail.next = null;
  newTab[j + oldCap] = hiHead;
}
```

就成下面这样了：

![image-20210406203927306](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406203927306.png)

然后就开始迁移

**低位迁移：首先：先将高位和低位断开（将低位的尾部的 next 置为空）**

```java
if (loTail != null) {
  // 把低位的尾部节点的 next 值为空（先将高位和低位断开）
  loTail.next = null;
  newTab[j] = loHead;
}
if (hiTail != null) {
  hiTail.next = null;
  newTab[j + oldCap] = hiHead;
}
```

就成下面这样了：

![image-20210406204245861](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406204245861.png)

紧接着将低位的头部 loHead 付给新数组的某个值，也就是将低位的所有节点移动过来，并且放在与就索引相同的位置：

```java
if (loTail != null) {
  // 把低位的尾部节点的 next 值为空（先将高位和低位断开）
  loTail.next = null;
  newTab[j] = loHead; // 将高位的头部赋给新数组的某个值，也就是将高位的所有节点移动过去
}
if (hiTail != null) {
  hiTail.next = null;
  newTab[j + oldCap] = hiHead;
}
```

![image-20210406204709734](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406204709734.png)

**高位迁移：先将高位的尾部断开**

```java
if (loTail != null) {
  // 把低位的尾部节点的 next 值为空（先将高位和低位断开）
  loTail.next = null;
  newTab[j] = loHead; // 将高位的头部赋给新数组的某个值，也就是将高位的所有节点移动过去
}
if (hiTail != null) {
  hiTail.next = null; // 把高位的尾部节点的 next 值为空
  newTab[j + oldCap] = hiHead;
}
```

再将高位的头部放到新数组的 j + oldCap 索引处（当前索引 + 旧数组的长度），比如说现在的索引是3，再加上数组长度16，最后就是将高位放到新数组的索引为19的地方去；

![image-20210406205220404](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406205220404.png)

然后我们发现这个设计非常的巧妙，避免了顺序的颠倒，再也不会出现狂神和佩雨颠倒的问题了

## 总结

在 JDK1.8 之后，HashMap 底层的数组扩容后迁移的方法进行了优化。把一个链表分成了两组，分成高位和低位分别去迁移，避免了死环问题。而且在迁移的过程中并没有进行任何的 rehash （重新计算 hash），提高了性能。它是直接将链表给断掉，进行几乎是一个均等的拆分，然后通过头指针的指向将整体给迁移过去，这样就减小了链表的长度
---
title: HashMap 数据结构及2的整数次幂探究
date: 2021-03-30
categories: [HashMap]
toc: true
---

## 1、数据结构

**JDK 1.7：数组 + 链表**

**JDK 1.8：hash 表 = 数组 + 链表 + 红黑树**



### 什么是哈希表？

哈希表（Hash table，也叫散列表），是根据关键码值（Key Value）而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找到速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。



## 2、HashMap 添加元素分析

在 HashMap 中，当添加元素时，会通过哈希值和数组的长度来计算得出一个`计算下标`，用它来准确的定位该元素应该 put 的位置。通常，我们为了使元素分布均匀会使用取模运算，用一个值去模上总长度，例如：

```java
index = hashCode % arr.length; // 实际并非这样
```

计算出 index 后，就会将该元素添加进去，理想状态下时将每个值都均匀的添加到数组中，但问题是不可能达到这样的理想状态，这时候就会产生哈希冲突，例如：`大狂神`通过计算添加到了数组3号位置；

![image-20210330210917396](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330210917396.png)

但是，此时`佩雨`这个元素通过计算产生了一个与大狂神相同的索引位置，这时就产生了**哈希冲突**；于是此时，就产生了第二种数据结构----链表，冲突的元素就会在该索引以链表的形式保存。

![image-20210330211132473](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330211132473.png)



## 3、了解：解决 Hash 冲突的四个方法

- 开放定址法（以冲突的下标为基础再次计算 hash，知道不冲突）

	这种方法也称再散列法，其基本思想是：当关键字 key 的哈希地址 `p = H (key)` 出现冲突时，以 p 为基础，产生另一个哈希地址 p1，如果 p1 仍然冲突，再以 p 为基础，产生另一个哈希地址 p2，。。。，直到找出一个不冲突的哈希地址 pi ，将相应元素存放入其中。

- 重 Hash 法（多个 hash 函数，一个 hash 函数计算出来的索引冲突了就会换一个函数计算，直到不冲突）

	这种方法是同时构造多个不同的哈希函数，当哈希地址 Hi = RH1（key）发生冲突时，再计算 Hi = RH2（key），直到冲突不再产生，这种方法不易产生聚集，但增加了计算时间。

- 链地址法

	这种方法的基本思想是将所有哈希地址为 i 的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第 i 个单位中，因而查找，插入和删除主要在同义词链中进行。链地址法适用于经常进行插入和删除的情况。

- 建立公共溢出区

	将哈希表分为公共表和溢出表，当溢出放生时，将所有溢出数据统一放到溢出区。



## 4、红黑树

之前的情况，当链表的长度过长时，其固有弊端就会显现出来，即查询效率较低，链表查询的时间复杂度是 O(n) ，数组的查询时间复杂度是 O(1)，链表查询效率非常低，所以就引出了第三种数据结构----**红黑树，链表长度 >= 8时链表转为红黑树**，红黑树是一颗接近于平衡的二叉树，其查询时间复杂度为 O(logn) ，远远比链表的查询效率高。

![image-20210330214431317](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330214431317.png)

**思考：那为什么要在链表长度大于等于8的时候变成红黑树呢？**

如果链表长度没有达到这个长度的话，因为红黑树它自身的这种维护，插入的这种维护的开销也是非常大的，因为每次去插入一个元素的时候很有可能会破坏掉他的平衡。也就是说 hashmap 的 put 操作非常多的时候，极有可能会影响插入的性能，因为插入一个元素的话，极有可能会打破它原有的平衡，那么每时每刻它都需要再恢复平衡（也就是红黑树的再平衡，需要左旋右旋，以及重新着色），就非常影响性能。



## 5、为什么数组的长度必须是2的指数次幂

**hashmap 中数组的初始长度（如果不传参，默认为16）**

```java
/**
 * The default initial capacity - MUST be a power of two.
 * 默认初始容量-必须是2的幂
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

**最大容量**：如果具有参数的任一构造函数隐式指定，则使用最大容量。必须是 2 的幂且 <= 1 << 30。

```java
/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the comstructors with arguments
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;
```

**如果我们传入的初始容量不是2的指数次幂，它就会将这个数改成大于该数且最接近这个数的2的指数次幂**

比如：

```java
HashMap<Object, Object> map = new HashMap<>(13);
map.put("allen", "Java");
```

​		传入的初始容量是13，它就会将13转换为大于13且最接近13的2的指数次幂的一个数；

```java
13 ------> 16
再比如：
7 ------> 8
5 ------> 8
3 ------> 4
```



## 6、源码分析

当我们使用构造方法传进来一个初始容量的值 `initialCapacity`，默认会传入一个加载因子 0.75；

```java
/**
 * Constructs an empty <tt>Hashmap</tt> with the specified initial 
 * capacity and the default load factor (0.75)
 */
public Hashmap(int initialcapacity){
  // 默认的加载因子仍是,75,其通过this关键字调用了本类的另外一个重方法
  this(initialcapacity, DEFAULT_LOAD_FACTOR);
}
```

继续追踪分析代码：

```java
public HashMap(int initialCapacity, float loadFactor) {
  			// 1.这里会先判断传来的初始容量是不是小于零的数字。如果是直接抛出异常
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
  			// 2.再判断是不是超过了hash定义的最大容量2的30次幕,如果超过了则让其等于最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
  			// 再接着判断传来的加载因子,如果小于零或者不是一个数字直接抛出异常
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
  			// 4.然后调用了一个 tablesizefor()方法去处理传进来的初始容量
        this.threshold = tableSizeFor(initialCapacity);
    }
```

继续追踪分析代码：进入到`tableSizeFor()`方法：这个处理初始容量的方法；

![image-20210330221047884](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330221047884.png)

​        解释：HashMap 为了实现存取高效,要尽量减少碰撞，就是要尽量做到：把数据分配均匀，保证每个链表长度大致相同，我们就需要一个算法来实现：将存入的数据保存到那个链表中的算法；而这个 算法实际就是取模：`hash % length`

​        但是，大家都知道这种运算不如位移运算快。因此，源码中做了优化`hash & (length - 1)`

​        也就是说`hash % length = hash & (length - 1)`

### 

### 那为什么是2的n次方呢？

因为2的n次方实际就是1后面n个0，而2的n次方-1，实际就是n个1。

例如长度为8时候，3&(8-1)=3  2&(8-1)=2，不同位置上，不碰撞，而长度为5的时候，3&(5-1)=0   2&(5-1)=0，都在0上，出现了碰撞。所以，保证容积是2的n次方，是为了保证在做(length - 1)的时候，每一位都能 &1，也就是和111...11111 进行与运算。**即：两位同时为“1”，结果才为“1”，否则为0**

**那么为什么要把初始容量转成2的指数次幂呢？不转成2度指数次幂也是可以存储的啊，为什么要转？**

首先看HashMap的put方法

```java
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, ture);	
}
```

其中的hash方法用于计算key的哈希值

```java
static final int hash(Object key){
  int h;
  return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



## 7、关于这个hash的思考

**为什么为什么要先高于16位或者低于16位再取模运算？**

hashmap这么做，只是为了避免降低hash冲突的机率。

打个比方，当我们的length为16的时候，哈希码（字符串“abcabcabcabcabc”的key对应的哈希码）和（16 - 1）进行与操作，对于多个key生成的hashCode，只要哈希码的后4位为0，不论高位怎么变化，最终的结果均为0，因为与运算中`两位同时为“1”，结果才为“1”，否则为0`；如下：

![image-20210330222816832](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330222816832.png)

而加上高16位移或低16位的“扰动函数”后，结果如下

![image-20210330222934222](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330222934222.png)

可以看到：扰动函数优化前：1954974080 % 16 = 1954974080 & （16 - 1）= 0；扰动函数优化后：1955003654 % 16 = 1955003654 & （16 - 1）=6；很显然，减少了碰撞的机率。

我们一开始提到过，添加元素索引的下标可以通过取模运算获得，但是我们知道计算机的运行效率：加法（减法）>乘法>除法>取模，取模的效率是最低的。所以我们要在HashMap中避免频繁的取模运算，又因为我们HashMap中他要通过取模去定位我们的索引，并且HashMap是在不停的扩容，数组一旦达到容量的阈值的时候就需要对数组进行扩容。那么扩容就意味着要进行数组的移动，数组一旦移动，每移动一次就要重回记算索引买这个过程中牵扯大量元素的迁移，就会大大影响效率。那么如果我们直接使用与运算，这个效率是远远高于取模运算的！



## 8、putVal方法，它是实现具体put操作的方法

```java
// 实现put操作
// 返回值：@return 之前的value
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
  			// 1.如果当前table为空，新建默认大小的table
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
  			// 2.获取当前key对应的节点
        if ((p = tab[i = (n - 1) & hash]) == null)
          	// 3.如果不存在，新建节点
            tab[i] = newNode(hash, key, value, null);
        else {
          	// 4.存在节点
            Node<K,V> e; K k;
          	// 5.key的hash相同，key的引用相同或者key equals，则覆盖
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
          	// 6.如果当前节点是一个红黑树树节点，则添加树节点
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
          	// 7.不是红黑树节点，也不是相同节点，则表示为链表结构
            else {
                for (int binCount = 0; ; ++binCount) {
                  	// 8.找到最后那个节点
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                      	// 9.如果链表长度超过8转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  	// 10.如果链表中有相同的节点，则覆盖
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
              	// 是否替换掉value值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
  			// 记录修改次数
        ++modCount;
  			// 是否超过容量，超过需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

由以上源码第二步，`tab[i = (n - 1) & hash]`中tab就是HashMap的实体数组，其下边通过`i = (n - 1) & hash`来获取（n表示数组长度，hash表示hashCode值），但是这必须保证数组长度为2的整数次幂。

现在我们可以使用**与运算**`(n - 1) & hash`取代取模运算`hash % length`，因为这两种方式计算出来的结果是一致的（n就是length），也就是`(length - 1) & hash = hash % length`，例如：加入数组长度为4，哈希值为10

```java
(n - 1) & hash = (4 -1) % 10 = 00000011 & 00001010 = 00000010 = 2;
hash % length = 10 % 4 = 2
```

显然，当数组长度不为2的整数次幂时二者是不相等的！

**但最重要的一点，是要保证定位出来的值是在数组的长度之内的，不能超出数组长度，并且减少哈希碰撞，让每个位都可能被取到**

![image-20210330231236782](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210330231236782.png)

**总结**：首先使用位运算来加快计算的效率，而要使用位运算们就需要数组-1然后与hash值保证其在数组范围内，只有当数组长度为2的指数次幂时，其计算得出的值才能和取模算法的值相等，并且保证能取到数组的每一位，减少哈希碰撞，不浪费大量的数组资源！


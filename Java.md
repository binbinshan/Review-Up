# Java


* [HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？](#1)

* [volatile 关键字解决了什么问题，它的实现原理是什么？](#2)


------

### <span id="1">1.HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？</span>

##### HashMap 与 ConcurrentHashMap 的实现原理是怎样的？
<details>
<summary>展开</summary>
在JDK1.7中，ConcurrentHashMap采用的是分段数组+链表的方式实现的，HashMap 则是采用数组+链表的方式实现的。

在JDK1.8中，ConcurrentHashMap采用的数据结构 和 HashMap的结构一样为 数组+链表/红黑二叉树。
</details>

##### ConcurrentHashMap 是如何保证线程安全的？
<details>
<summary>展开</summary>
JDK 1.7 中使用分段锁，一个 ConcurrentHashMap 里包含一个 Segment 数组。Segment 的结构和 HashMap 类似，是一种数组和链表结构，一个 Segment 包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素，每个 Segment 守护着一个 HashEntry 数组里的元素，当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment 的锁。
  
JDK 1.8 中，取消类 Segment 分段锁，采用Node + CAS + Synchronized来保证并发安全。synchronized 只锁定当前链表或红黑二叉树的首节点。
当 HashEntry 对象组成的链表长度超过 TREEIFY_THRESHOLD 时，链表转换为红黑树，提升性能。底层变更为数组 + 链表 + 红黑树
</details>


### <span id="2">2.volatile 关键字解决了什么问题，它的实现原理是什么？</span>

##### 解决了什么问题
<details>
<summary>展开</summary>
volatile 解决了可见性、防止重排序的问题。

* 可见性：可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区。volatile关键字能有效的解决这个问题。
* 防止重排序：操作系统可以对指令进行重排序，volatile可以防止重排序。
</details>

##### 实现原理
<details>
<summary>展开</summary>

* 可见性的实现原理：
被volatile修饰的变量会存在一个“:Lock”前缀，Lock可以理解为CPU指令集的一种锁，同时该指令会将当前处理器缓存行的数据直接写入系统内存中，写的操作会导致其他CPU里缓存了该地址的数据无效，当处理器发现本地缓存失效后，就会从内存中获取最新的值，这样就保证了变量的可见性。

* 防止重排序的实现原理：
JMM为了优化性能，在不改变正确原意的情况下，允许编译器和处理器对指令进行重排序。
volatile防止重排的实现是基于happens-before，其中happens-before有一条规则，所有的volatile域的读应该在volatile域的写之后。为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
</details>
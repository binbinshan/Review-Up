# Java


* [HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？](#1)




------

### <span id="1">HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？</span>

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

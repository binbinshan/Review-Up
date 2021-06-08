# Java


* [HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？](#1)

* [volatile 关键字解决了什么问题，它的实现原理是什么？](#2)

* [Synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么?](#3)

* [Java 中垃圾回收机制中如何判断对象需要回收？常见的 GC 回收算法有哪些？](#4)


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


### <span id="3">3.Synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么？</span>

##### 实现原理
<details>
<summary>展开</summary>
1. synchronized修饰的同步语句块，实现是使用monitorenter和monitorexit指令，其中monitorenter指向同步代码块开始位置，monitorexit指向同步代码块结束的位置。

2. synchronized修饰的方法，则是用acc_synchronized 来标记，该标识指明了该方法是一个同步方法。
3. 两者的本质都是对对象监视器 monitor 的获取。
</details>

##### Synchronized执行流程
<details>
<summary>展开</summary>
1. 判断Mark word里是否是当前线程，且偏向锁状态为true，如果是，则获得偏向锁。

3. 如果不是，那么执行CAS将Mark work修改为当前线程，如果成功，则获得偏向锁，并设置偏向锁为true。否则发生竞争，撤销偏向锁，升级为轻量级锁。
4. 当前线程执行CAS，将mark word替换为锁记录指针。如果成功获得轻量级锁。如果失败，当前线程尝试自旋获取锁，如果获取到，处于轻量级锁。如果一定次数后没获取到锁，锁升级为重量级锁。
5. 重量级锁是独占锁。通过monitor对象实现。monitor enter 进入数+1，当前线程获得锁，如果其他线程已经获得锁，那么该线程阻塞，直到锁计数为0。monitor exit表示锁计数-1，直到进入数为0时，当前线程释放锁。

</details>

##### 与 ReentrantLock 相比优缺点分别是什么？
<details>
<summary>展开</summary>
1. 实现：synchronized 是 JVM 实现的，是一个关键字 ，Lock 是 接口，基于JDK实现的；

3. 性能：JDK1.6后对synchronized优化后，两者性能差距不大。
4. 锁：synchronized是非公平锁，而ReentrantLock既可以是公平也可以是非公平。
5. 绑定锁的条件：synchronized只支持一个条件，ReentrantLock则可以支出多个条件。

除非要使用ReentrantLock的高级功能外，基于都是用synchronized，因为synchronized会自动释放锁，不会导致死锁，并且是jvm原生支持的。

</details>


### <span id="4">4.Java 中垃圾回收机制中如何判断对象需要回收？常见的 GC 回收算法有哪些？</span>

##### 如何判断对象需要回收
<details>
<summary>展开</summary>
JVM判断对象是否需要被回收，主要是有两种方法:
* 引用计数法
给对象添加一个引用计数，当被引用时引用计数+1，当引用失效时，引用计数-1。当引用计数为0的时候，代表该对象可以被回收。
但是引用计数法有一个问题，就是无法解决相互依赖，两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。 正因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法。

* 可达性分析算
通过GC root作为起始点往下搜索，能够到达的对象都是不需要回收的，不能到达的需要被回收。
GC Root一般包含以下内容：
    * 虚拟机栈引用的对象
    * 本地方法栈引用的对象
    * 方法区中静态属性引用的对象
    * 方法区中常量引用的对象
</details>

##### 常见的 GC 回收算法有哪些?
<details>
<summary>展开</summary>
* 标记-清除算法
将存活的对象进行标记，会被标记的会被清除
缺点：标记和清除的效率不高，且会产生大量的内存随便，导致无法创建大对象。
* 标记-整理算法
将存活的对象进行标记，然后将存活的对象移动到一端，然后把剩余的空间清除。

* 复制算法
将内存分为两块，每次只是用一块，当该块内存满了之后，就复制到另一块内存上，将该内存块进行清理。主要的缺点就是只能使用内存的一部分。
就比如Hotspot中就将新生代分为了三部分，一个Eden区和两个Survivor区，每次只使用E区和一个S区，当进行回收的时候，就会把将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和使用过的那一块 Survivor。 
HotSpot 虚拟机的 Eden 和 Survivor 的大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 空间就不够用了，此时需要依赖于老年代进行分配担保，也就是借用老年代的空间存储放不下的对象。 

* 分代回收算法
现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。 一般将堆分为新生代和老年代。 
新生代：复制算法
老年代：标记清除、标记整理
</details>
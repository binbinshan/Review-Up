# Java


* [HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？](#1)

* [volatile 关键字解决了什么问题，它的实现原理是什么？](#2)

* [Synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么?](#3)

* [Java 中垃圾回收机制中如何判断对象需要回收？常见的 GC 回收算法有哪些？](#4)

* [集合类中的 List 和 Map 的线程安全版本是什么，如何保证线程安全的？](#5)

* [ThreadLocal 实现原理是什么？](#6)

* [String 类能不能被继承？为什么？](#7)


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


### <span id="5">5.集合类中的 List 和 Map 的线程安全版本是什么，如何保证线程安全的？</span>
<details>
<summary>展开</summary>
在平常开发中，经常使用的List 就是 ArrayList 和 LinkedList，这两个都不是线程安全的。经常使用的Map 就是HashMap，也不是线程安全的。

所以在多线程并发下，这些就不能使用了，必须使用线程安全的容器。

### List
1. 使用Collections.synchronizedList(List<T> list) 可以将其包装成一个线程安全的 List。
2. Vector：在它的大部分方法上添加了 synchronized 关键字，用来保证线程安全。

### Map
1. 使用Collections.synchronizedMap(Map<T> map) 可以将其包装成一个线程安全的 Map。
2. HashTable：在它的大部分方法上添加了 synchronized 关键字，用来保证线程安全。HashTable 的 Key 和 Value 都不允许为 null。
3. ConcurrentHashMap ：1.8之前采用分段锁，1.8之后取消分段锁，采用Node + CAS + Synchronized来保证并发安全

### Set
1. 使用Collections.synchronizedSet(Set<T> set) 可以将其包装成一个线程安全的 Set。
2. ConcurrentSkipListSet


### Queue
1. ConcurrentLinkedQueue：通过无锁的方式(CAS)，实现了高并发状态下的高性能
2. ConcurrentLinkedDeque：通过无锁的方式(CAS)，实现了高并发状态下的高性能
3. LinkedBlockingDeque：一个线程安全的双端队列实现。它的内部使用链表结构，每一个节点都维护了一个前驱节点和一个后驱节点。
</details>

### <span id="6">6.ThreadLocal 实现原理是什么？</span>

##### 实现原理
<details>
<summary>展开</summary>
ThreadLocal为每个线程都提供了变量的副本，使得每个线程在某一时间访问到的并非同一个对象，这样就隔离了多个线程对数据的数据共享。

实现原理：
1. 每个线程都会有一个变量threadLocals,这个threadLocals就是通过ThreadLocal进行维护的ThreadLocalMap，
2. 这个map是key-val结构的；Map里面的key为ThreadLocal的弱引用，value为需要共享的值。所以每个线程只能拿到根据自己创建的ThreadLocal去ThreadLocalMap中获取对象，到达了线程隔离的效果。
</details>

##### ThreadLocal内存泄露
<details>
<summary>展开</summary>

1. ThreadLocal是被ThreadLocalMap以弱引用的方式关联着。

3. 因此如果ThreadLocal没有被ThreadLocalMap以外的对象引用，则在下一次GC的时候，ThreadLocal实例就会被回收，那么此时ThreadLocalMap里的一组KV的K就是null了
4. 因此在没有额外操作的情况下，此处的V便不会被外部访问到，而且只要Thread实例一直存在，Thread实例就强引用着ThreadLocalMap，因此ThreadLocalMap就不会被回收
5. 这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value，而这块value永远不会被访问到了，所以存在着内存泄露。

注意：如果频繁的在线程中new ThreadLocal对象，在使用结束时，最好调用ThreadLocal.remove来释放其value的引用，避免在ThreadLocal被回收时value无法被访问却又占用着内存

</details>


### <span id="7">7.String 类能不能被继承？为什么？</span>

##### 实现原理
<details>
<summary>展开</summary>
String类是被final关键字修饰了，所以无法被继承。
因为将引用声明作final，就不能改变这个引用了，编译器会检查代码，如果试图将变量再次初始化的话，编译器会报编译错误。

对于 final 域，编译器和处理器要遵守两个重排序规则：
1. 在构造函数内对一个 final 域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
2. 初次读一个包含 final 域的对象的引用，与随后初次读这个 final 域，这两个操作之间不能重排序。

写 final 域的重排序规则：
编译器要求在 写final域 之后，构造函数return之前插入一个StoreStore障屏，确保屏障前的写操作在屏障后的写操作之前。

读 final 域的重排序规则：
编译器要求在 读final域 之前，插入一个LoadLoad屏障，确保屏障前的读操作在屏障后的读操作之前。
</details>

##### 我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?
<details>
<summary>展开</summary>

因为String类是final，所以我们无法继承重写，但是可以使用组合的方式来实现，
```
    class MyString{
    
        private String innerString;
    
        // ...init & other methods
    
        // 支持老的方法
        public int length(){
            return innerString.length(); // 通过innerString调用老的方法
        }
    
        // 添加新方法
        public String toMyString(){
            //...
        }
    }
```
</details>
# Java

     
   ```
   tips:如需要展开全部,可以在网页控制台执行以下代码
   [...document.getElementById("readme").getElementsByTagName("details")].forEach(e => e.open = true)
   ```

* [HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？](#1)

* [volatile 关键字解决了什么问题，它的实现原理是什么？](#2)

* [Synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么?](#3)

* [Java 中垃圾回收机制中如何判断对象需要回收？常见的 GC 回收算法有哪些？](#4)

* [集合类中的 List 和 Map 的线程安全版本是什么，如何保证线程安全的？](#5)

* [ThreadLocal 实现原理是什么？](#6)

* [String 类能不能被继承？为什么？](#7)

* [== 和 equals() 的区别？](#8)

* [JMM 中内存模型是怎样的？什么是指令序列重排序？](#9)

* [Java 线程和操作系统的线程是怎么对应的？Java线程是怎样进行调度的? ](#10)

* [简述 Spring bean 的生命周期](#11)

* [简述 Spring 的 IOC 机制](#12)

* [简述 Spring 注解的实现原理](#13)

* [简述 Spring AOP 的原理 ](#14)

* [Spring是如何解决循环依赖的？](#15)

* [Spring的Bean作用域有哪些？](#16)

* [HashMap的知识点](#17)



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

------ 
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

------ 

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

------ 

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

------ 

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

------ 

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

------ 

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

------ 

### <span id="8">8.== 和 equals() 的区别？</span>

<details>
<summary>展开</summary>

== : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

* 情况 1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
* 情况 2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。
</details>

------ 

### <span id="9">9.JMM 中内存模型是怎样的？什么是指令序列重排序？</span>

##### JMM 中内存模型是怎样的？
<details>
<summary>展开</summary>

![](https://github.com/binbinshan/Review-Up/blob/master/images/Java/16164236743234.jpg)

每个线程都有自己的工作内存，是CPU级别的缓存：
1. 首先从主内存中read数据，然后load到工作内存中
2. 线程use数据，进行操作
3. 线程assign(赋值)到自己的工作内存中
4. 工作内存再去stroe(存储)变量 write到主内存中。

这也就解释了多线程操作同一个数据可能出现的问题 ，主要就是因为把变量保存本地内存中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。

</details>

##### 什么是指令序列重排序？
<details>
<summary>展开</summary>
JMM为了优化性能，在不改变正确原意的情况下，允许编译器和处理器对指令(代码)进行重排序。
在有些情况下需要代码不能进行重排，必须保证其的有序性。可以使用volatile关键字保证有序性。
</details>

------ 

### <span id="10">10.Java 线程和操作系统的线程是怎么对应的？Java线程是怎样进行调度的? </span>

##### Java 线程和操作系统的线程是怎么对应的？
<details>
<summary>展开</summary>

###### 线程实现
Java线程是用本地线程实现的，所以使用线程的Java程序与使用线程的本地程序没有区别。“Java 线程”只是属于 JVM 进程的线程。

###### 线程状态
* 操作系统中的线程，只有ready、running、waiting三种状态
    * ready:表示线程已经被创建，正在等待系统调度分配CPU使用权
    * running:表示线程获得了CPU使用权，正在进行运算
    * waiting:表示线程等待，让出CPU资源给其他线程使用

* Java线程状态存在6种，NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED
    * NEW、TERMINATED 这两种状态实际上并不属于系统线程的运行状态，是Java线程独有的状态。
    * RUNNABLE 对应操作系统线程中的 ready、running
    * BLOCKED、WAITING、TIMED_WAITING 对应操作系统中 waiting
</details>

##### 系统线程的调度方式?
<details>
<summary>展开</summary>

线程调度是指系统为线程分配处理器使用权的过程，主要调度方式分两种，分别是：
* 协同式线程调度 ：线程执行时间由线程本身来控制，线程把自己的工作执行完之后，要主动通知系统切换到另外一个线程上。最大好处是实现简单，且切换操作对线程自己是可知的，没有同步问题，坏处是线程执行时间不可控制，如果一个线程有问题，可能一直阻塞在那里。

* 抢占式线程调度：每个线程将由系统来分配执行时间，线程的切换不由线程本身来决定（Java中，Thread.yield()可以让出执行时间，但无法获取执行时间）。线程执行时间系统可控，也不会有一个线程导致整个进程阻塞。
</details>

##### Java线程是怎样进行调度的?
<details>
<summary>展开</summary>

Java线程的调度模式使用的抢占式线程调度。
如果希望系统能给某些线程多分配一些时间，给一些线程少分配一些时间，可以通过设置线程优先级来完成，Java语言一共10个级别的线程优先级，在两线程同时处于ready状态时，优先级越高的线程越容易被系统选择执行。
但优先级并不是很靠谱，因为Java线程是通过映射到系统的原生线程上来实现的，所以线程调度最终还是取决于操作系统。
</details>




------ 

### <span id="11">11.简述 Spring bean 的生命周期</span>

<details>
<summary>展开</summary>

# 简述 Spring bean 的生命周期

![](https://github.com/binbinshan/Review-Up/blob/master/images/Java/16260070649694.jpg)

Bean 的生命周期概括起来就是 4 个阶段：
* 实例化（Instantiation）

* 属性赋值（Populate）

* 初始化（Initialization）

* 销毁（Destruction）


### 详细步骤

* 实例化：第 1 步，实例化一个 bean 对象；

* 属性赋值：第 2 步，为 bean 设置相关属性和依赖；

* 初始化：第 3~7 步，步骤较多，其中第 5、6 步为初始化操作，第 3、4 步为在初始化前执行，第 7 步在初始化后执行，该阶段结束，才能被用户使用；

* 销毁：第 8~10步，第8步不是真正意义上的销毁（还没使用呢），而是先在使用前注册了销毁的相关调用接口，为了后面第9、10步真正销毁 bean 时再执行相应的方法。

具体步骤：
1. Spring对Bean进行实例化（相当于程序中的new Xx()）

2. Spring将值和Bean的引用注入进Bean对应的属性中

3. 如果Bean实现了Aware接口，就能在 bean 中获得相应的 Spring 容器资源。比如bean的id属性、当前BeanFactory容器的引用等。

4. 如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessBeforeInitialization（初始化前置处理）方法（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）

5. 如果Bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet方法，作用是在Bean的全部属性设置成功后执行的初始化方法。

6. 在配置文件中对Bean使用init-method声明初始化，与实现InitializingBean接口的作用一样，作用都是在Bean的全部属性设置成功后执行的初始化方法，

7. 如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessAfterInitialization（初始化后置处理）方法（作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 )

8. 经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁

9. 如果Bean实现了DispostbleBean接口，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。



### 总结
Spring Bean 的生命周期：

1. 首先是实例化、属性赋值、初始化、销毁这 4 个大阶段；

3. 然后初始化的具体操作，有 Aware 接口的依赖注入、BeanPostProcessor 在初始化前后的处理以及 InitializingBean 和 init-method 的初始化操作；

4. 销毁的具体操作，有注册相关销毁回调接口，最后通过DisposableBean 和 destory-method 进行销毁。

</details>



------ 

### <span id="12">12.简述 Spring 的 IOC 机制</span>

<details>
<summary>展开</summary>


IOC叫做控制反转，是一种设计思想，意味着你将设计好的对象交给容器控制，而不是在对象内部直接创建。

* 谁控制谁，控制了什么？

> 首先解释下控制是什么意思？控制就是对象创建、初始化、销毁。

创建对象：原来创建对象是通过new一下，现在spring容器给创建了。

初始化：原来通过构造器或者setter方法赋值，现在 Spring容器给自动注入了

销毁：之前给对选哪个赋值null或者销毁操作，现在spring容器负责销毁。

IOC解决了繁琐的管理对象声明周期的操作，解耦了我们的代码。

* 为什么是反转，反转了些什么

传统应用程序中，通过引入对象来主动获取依赖对象，这是正转。

Spring中，通过容器来创建和注入对象，对象只是被动的接受依赖对象，依赖对象的获取方式被容器反转了，这是反转。

反转后，我们无法决定对象生命周期的任何阶段，最多借助spring的扩展做一些小动作。

</details>



------ 

### <span id="13">13.简述 Spring 注解的实现原理</span>

<details>
<summary>展开</summary>

1. Spring如何使用注解机制完成自动装配
Java实例构造时会调用默认父类无参构造方法，Spring正是利用了这一点，让"操作元素的代码"得以执行。

【两种处理策略】
1. 类级别的注解：如@Component、@Repository、@Controller、@Service以及JavaEE6的@ManagedBean和@Named注解，都是添加在类上面的类级别注解。
Spring容器根据注解的过滤规则扫描读取注解Bean定义类，并将其注册到Spring IoC容器中。

2. 类内部的注解：如@Autowire、@Value、@Resource以及EJB和WebService相关的注解等，都是添加在类内部的字段或者方法上的类内部注解。
SpringIoC容器通过Bean后置注解处理器解析Bean内部的注解。


注解注入在 XML 注入之前执行。因此，XML 配置会覆盖通过这两种方法连接的属性的注释。

</details>



------ 

### <span id="14">14.简述 Spring AOP 的原理 </span>

<details>
<summary>展开</summary>

Spring AOP的实现是采用动态代理。

* Spring AOP 默认为 AOP 代理使用标准的 JDK 动态代理。这允许代理任何接口。

* Spring AOP 也可以使用 CGLIB 代理。默认情况下，如果业务对象未实现接口，则使用 CGLIB。



### JDK动态代理
动态代理(JDK代理/接口代理)，使用的JDK API，需要目标对象实现了接口才可以。(因为JDK代理需要用构造方法动态获取具体的接口信息，如果不实现接口的话，没法初始化）
JDK中生成代理对象主要涉及的类有java.lang.reflect Proxy，主要方法为
```
//返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h ) 

 ClassLoader loader    //指定当前目标对象使用类加载器
 Class<?>[] interfaces //目标对象实现的接口的类型
 InvocationHandler h   //事件处理器（下方处理方法）
```
java.lang.reflect InvocationHandler，主要方法为
```
// 在代理实例上处理方法调用并返回结果。
Object invoke(Object proxy, Method method, Object[] args) 
```

### CGLIB动态代理
cglib (Code Generation Library )是一个第三方代码生成类库，CGLIB会让生成的代理类继承被代理类，并在代理类中对代理方法进行强化处理，使用cglib代理的对象则无需实现接口，但是无法代理被final修饰的类。
代理类将委托类作为自己的父类，并为其中的非final委托方法创建两个方法：
* 一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；
* 另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理。

底层将方法全部存入一个数组中，通过数组索引直接进行方法调用。



</details>



------ 

### <span id="15">15.Spring是如何解决循环依赖的？</span>

<details>
<summary>展开</summary>

Spring中的依赖注入有两种形式：

* 构造函数的依赖注入
* Setter的依赖注入

Spring官方文档上写到The Spring team generally advocates constructor injection**(Spring团队通常支持构造函数注入)**，但是只有注入方式是**setter**且**singleton** ，才不会有循环依赖问题。

解决循环依赖主要是利用三级缓存：
```
 //一级缓存
	/** Cache of singleton objects: bean name to bean instance. */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
//三级缓存
	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
//二级缓存
	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

```

* 第一级缓存（也叫单例池）singletonObjects：存放已经经历了完整生命周期的Bean对象。

* 第二级缓存：earlySingletonObjects，存放早期暴露出来的Bean对象，Bean的生命周期未结束（属性还未填充完）。

* 第三级缓存："Map<String, ObjectFactory<?>>" singletonFactories，存放可以生成Bean的工厂。

所以Spring解决循环依赖依靠的是Bean的"中间态"这个概念，具体是依赖三级缓存来实现的。

### 核心逻辑
假设A、B循环引用：
* 实例化A的时候就将其放入三级缓存中，接着属性赋值的时候，发现依赖了B

* 同样的流程将B实例化后放入三级缓存，接着去填充属性时又发现B依赖A

* B先查一级缓存，没有A，再查二级缓存，还是没有A，再查三级缓存，找到了A然后把三级缓存里面的这个A，如果没有AOP代理的话，直接将A的原始对象注入B，如果有AOP代理，就进行AOP处理获取代理后的对象A，然后将A放到二级缓存里面，并删除三级缓存里面的A。

* 完成B的初始化后，进行属性填充和初始化，这时候B完成后，就去完成剩下的A的步骤，此时B已经创建结束，直接从一级缓存里面拿到B，然后完成创建，并将A自己放到一级缓存里面。


### 代码实现
1. 调用doGetBean()方法，想要获取beanA，于是调用getSingleton()方法从缓存中查找beanA

3. 在getSingleton()方法中，从一级缓存中查找，没有，返回null
4. doGetBean()方法中获取到的beanA为null，于是走对应的处理逻辑，调用getSingleton()的重载方法（参数为ObjectFactory的)
5. 在getSingleton()方法中，先将beanA_name添加到一个集合中，用于标记该bean正在创建中。然后回调匿名内部类的creatBean方法
6. 进入AbstractAutowireCapableBeanFactory#ndoCreateBean，先反射调用构造器创建出beanA的实例，然后判断:是否为单例、是否允许提前暴露引用(对于单例一般为true)、是否正在创建中（即是否在第四步的集合中）。判断为true则将beanA添加到【三级缓存】中
7. 对beanA进行属性填充，此时检测到beanA依赖于beanB，于是开始查找beanB
8. 调用doGetBean()方法，和上面beanA的过程一样，到缓存中查找beanB，没有则创建，然后给beanB填充属性
9. 此时 beanB依赖于beanA，调用getSingleton()获取beanA，依次从一级、二级、三级缓存中找，此时从三级缓存中获取到beanA的创建工厂，通过创建工厂获取到singletonObject，此时这个singletonObject指向的就是上面在doCreateBean()方法中实例化的beanA
10. 这样beanB就获取到了beanA的依赖，于是beanB顺利完成实例化，并将beanA从三级缓存移动到二级缓存中
11. 随后beanA继续他的属性填充工作，此时也获取到了beanB，beanA也随之完成了创建，回到getsingleton()方法中继续向下执行，将beanA从二级缓存移动到一级缓存中


</details>


------ 

### <span id="16">16.Spring的Bean作用域有哪些？</span>

<details>
<summary>展开</summary>

Spring Bean 中所说的作用域，在配置文件中即是“scope”，在面向对象程序设计中作用域一般指对象或变量之间的可见范围。

而在Spring容器中是指其创建的Bean对象相对于其他Bean对象的请求可见范围。

* Singleton: Spring IOC容器中仅存在一个Bean，单例的形式，Bean的缺省作用域。

* Prototype: 每次从容其中调用Bean的时候都会创建一个Bean。

* Request: 每次Http请求都创建一个Bean。

* Session: 每个用户Session可以产生一个新的Bean，不同用户之间的Bean互不影响。

* application:仅在ServletContext的生命周期内有效。

* websocket:将单个 bean 定义范围限定为WebSocket。


另外在加载对象的时机上分为懒汉式和饿汉式：

* 懒汉式：Spring默认是不启用 懒加载的，属性是 ”lazy-init“ 的bean Spring初始化阶段不会进行init并且依赖注入，当第一次进行getBean时候进行初始化并依赖注入。

* 饿汉式：在进行getBean的时候会从缓存里取，因为容器初始化阶段已经初始化了。

* 单例模式下的Bean存在懒汉式和饿汉式，而非单例的Bean则是自动启动懒加载模式，没有饿汉模式。



</details>



------ 

### <span id="17">17.HashMap的知识点</span>

<details>
<summary>展开</summary>

1. HashMap的底层数据结构？

JDK1.7中，HashMap采用的数据结构为数组+链表。如果哈希冲突时，插入链表是采用头插法。

JDK1.8中，HashMap采用的数据结构为数组+链表/红黑二叉树。如果哈希冲突时，插入链表是采用尾插法。

2. 为什么要使用尾插法？

使用头插会改变链表的上的顺序，但是如果使用尾插，在扩容时会保持链表元素原本的顺序，就不会出现链表成环的问题了。

即使JDK1.8中HashMap的不会因为扩容导致死循环，但是也不适合在多线程中使用，因为put/get方法都没有加同步锁。

3. 为什么使用红黑树呢？

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构，这样遍历的时间复杂度和链表一样都是O(N)。

而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，

我们知道红黑树属于平衡二叉树，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。


4. 为什么会线程不安全？

在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失。

在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。

5. 有什么线程安全的类代替么?
可以使用Hashtable、CurrentHashMap。

6. 默认初始化大小是多少？为什么大小都是2的幂？

默认初始化大小是16，这是为了实现Hash的均匀分布。

HashMap没有采用常见对hash值进行取模的方式计算地址，而是采用了Hash值和(数组长度-1)进行与运算，首先两种方式的效果是一样的，这是因为HashMap中数组的长度始终是2的平方。采用与运算的原因是效率更高，

7. HashMap的扩容方式？负载因子是多少？为什是这么多？
当HashMap中数组元素的数量个数达到了 HashMap * 负载因子，默认值0.75f, 就进行扩容。比如有一个16大小的HashMap，当数组里面要放第13个元素的时候，就会进行扩容。

扩容分为两步：

    1. 扩容：创建一个新的Entry空数组，长度是原数组的2倍。

    2. reHash：遍历原Entry数组，把所有的Entry重新Hash到新数组。

reHash 需要把hash值和扩大后的数组长度-1 与运算 ----> (HashCode（Key） & （Length - 1）)。

这里需要注意，当链表的长度到达8时，会转换为红黑树，但是这里如果当前数组小于64，就会先扩容数组。

8. HashMap是怎么处理hash碰撞的？

hash碰撞是因为还是可能会hash值一样，导致出现地址相同的情况，所以HashMap会采用拉链法解决，形成一个链表；

当链表的长度达到8的时候，就会扩展为红黑树。当长度小于等于6的时候，就会退化为链表。

9. hash的计算规则？

HashMap 把 hashCode()计算出的值进行了优化，将hash值 与 hash值右移16位的hash值 进行异或，这样hash值高低位就都参与了计算。


10. 为啥我们重写equals方法的时候需要重写hashCode方法呢？

在没有覆盖equals方法的情况下，equals和 == 一样，对基本类型比较的是值，对引用对象比较的是内存地址。

如果只重写equals(例如根据对象的字段判断是否相同)，却没有重写hashCode，则可能会导致两个字段值相同的对象equals是相同的，但是hashCode是不同的。违背了equals 为 true ， hashCode 必须相等。

例如在HashMap中使用自定义对象作为key的时候，如果不重写equles和hashCode方法，那么这个时候正常使用是没有问题的;

假设只重写了自定义对象的equals方法，却没有重写hashCode方法，就会导致hash值不一样，那么在put时，HashMap里面本来有这个key，但是因为hash不一样，导致了put操作成功。逻辑上是不符合规范的，get时取出来的也可能是自己另一个的value。

</details>


* [能简单介绍下Linux中的top命令吗？](#1)









------

### <span id="1">1.能简单介绍下Linux中的top命令吗？</span>

<details>
<summary>展开</summary>

top命令是Linux下常用的**实时**的性能分析工具，能够实时的显示系统中各个进程占用的资源，类似于windows中的任务管理器。

<div align="center"> 
    <img src="https://github.com/binbinshan/Review-Up/blob/master/images/Linux/top1.png" width="1362px"> 
</div>
<br>

上图中，共分为两部分，第一部分为系统总体角度，第二部分为每个进程部分。


上图中，共分为两部分，第一部分为系统总体角度(前五行)，第二部分为每个进程部分。

第一行：系统级信息
* 21:43:25 代表当前时间
* up 257 days, 8:21 代表服务器开机至现在的运行时长
* 4 users 当前在线用户数量
* load average: 1.02, 1.18, 1.20： 系统1分钟、5分钟、15分钟的CPU负载信息

第二行：任务
* 305 total 当前有305个任务，也就是305个进程
* 3 running 3个进程正在运行
* 302 sleeping 302个进程正在休眠
* 0 stopped 0个停止的进程
* 0 zombie  0个僵尸进程

第三行：cpu
* 6.7%us  用户态进程占用CPU时间百分比，不包含renice值为负的任务占用的CPU的时间。
* 7.0%sy  内核态占用CPU时间百分比
* 0.0%ni  改变过优先级的进程占用CPU的百分比
* 86.3%id 空闲CPU时间百分比
* 0.0%wa：等待I/O的CPU时间百分比
* 0.0%hi：CPU硬中断时间百分比
* 0.0%si：CPU软中断时间百分比

第四行：内存
* 32779248k total 物理内存总量32G
* 2705880k used 使用的物理内存量2.5G
* 9943348k free 空闲的物理内存量9.4G
* 20130020k buffers 用作内核缓存的物理内存量19G

第五行：交换空间
* 4194300k total：交换区总量
* 4032104k used：使用的交换区量
* 162196k free：空闲的交换区量
* 15860624k cached：缓冲交换区总量

每个进程的信息：
* PID：进程的ID
* USER：进程所有者
* PR：进程的优先级别，越小越优先被执行
* NI：值
* VIRT：进程占用的虚拟内存
* RES：进程占用的物理内存
* SHR：进程使用的共享内存
* S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
* %CPU：进程占用CPU的使用率
* %MEM：进程使用的物理内存和总内存的百分比
* TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
* COMMAND：进程启动命令名称


</details>

------ 
---
title: 操作系统高频面试题
excerpt: 关于操作系统的高频面试题，内容都不深，前端最好也掌握
date: 2022-09-30
categories:
- 技术文章
tags:
- 操作系统
- 面试
---

## 为什么有了进程，还要有线程呢？
为了提高系统资源的利用率和系统的吞吐量，通常进程可让多个程序并发的执行，但是也会带来一些问题

- 进程如果在执行的过程被阻塞，那这个进程将被挂起，这时候进程中有些等待的资源得不到执行：
- 进程在同一时间只能做一件事儿

基于以上的缺点，操作系统引入了比进程更小的线程，作为并发执行的基本单位，从而减少程序在并发执行时所付出的时间和空间开销，提高并发性能。

**举个栗子**
开发一款聊天软件，A给B发消息，一直没收到，单步调试发现程序卡在了wati for user input不动了。解决方法是设置个1s，用户1s不输入直接跳过进行后面的接收阶段和显示阶段，但这样用户输入信息就很可能丢失。

于是又将输入和显示这两个动作给分开，一个负责输入，发送消息，一个负责读信息和显示，这就是我们的多进程。不过多进程也还是有些问题需要注意，开多个窗口没问题，无脑开窗口容易内存耗尽，而且想要几个窗口交换个数据也是非常麻烦。这是因为多进程的程序，每个进程都有自己的独立内存空间，不能相互乱看，要想通信就要接触系统层面来通信，所以肯定就会造成较多的资源消耗和时间浪费。几个进程为了方便，干脆开辟一块内存空间，共乐其中，这就是线程非常重要的意义，不过共享了不代表我们就是"裸"的，个人保密还是要做到，也不要吵架，至于通过锁的方式保密，就涉及到了线程的同步。

## 简单说下你对并发和并行的理解？
**并发**
- 在一个时间段中多个程序都启动运行在同一个处理机中

**并行**
- 假设目前A，B两个进程，两个进程分别由不同的CPU管理执行，两个进程不抢占CPU资源且可以同时运行，这叫做并行。

**举个栗子**
并发是指多个任务在一段时间内发生。比如火锅店做活动，只有200个位置，但是来的人太多了，来了250个人，此时多出的50个人只好等待着或者去另外家火锅店。那么火锅店老板对这250的安排不是同一时刻安排而是一段时间去处理，其实这就是并发。

![](https://api2.mubu.com/v3/document_image/97aba1a2-c6a3-4b32-ab54-b4a622d7236e-3807603.jpg)

平时玩儿电脑的时候，边写代码边听音乐，计算机同时处理了这么多任务。如果是单核CPU，在我们看来这些事儿是同时发生的，其实那是因为底层CPU切换的速度太快以致于我们完全感受不到它的切换，仅此为错觉而已。但是如果是多核CPU，各个CPU负责不同的进程，各个进程不抢占CPU，这样同时进行，这就是真正意义上的并行。

![](https://api2.mubu.com/v3/document_image/796de415-fb55-4dea-bf4e-e9bb20d31b16-3807603.jpg)

两者区别在于是否"同时"发生。是在一段时间同时发生还是多个事情在同一个时间点同时发生。

## 同步、异步、阻塞、非阻塞的概念
首先大家应该知道同步异步，阻塞非阻塞是两个不同层面的问题，一个是operation层面，一个是kernal层面。同步异步最大的区别在于是否需要底层的响应再执行。阻塞非阻塞最大的区别在于是否立即给出响应。

**同步**：当一个同步调用发出后，调用者要一直等待返回结果。通知后，才能进行后续的执行。

**异步**：当一个异步过程调用发出后，调用者不能立刻得到返回结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。

**阻塞**：是指调用结果返回前，当前线程会被挂起，即阻塞。

**非阻塞**：是指即使调用结果没返回，也不会阻塞当前线程。

**举个栗子**
- 小Q去钓鱼，抛完线后就傻傻的看着有没有动静，有则拉杆(同步阻塞)
- 小Q去钓鱼，拿鱼网捞一下，有没有鱼立即知道，不用等，直接就捞(同步非阻塞)
- 小Q去钓鱼，这个鱼缸比较牛皮，扔了后自己就打王者荣耀去了，因为鱼上钩了这个鱼缸带的报警器会通知我。这样实现异步(异步非阻塞）

## 进程和线程的相关题
- **进程**：进程是系统进行资源分配和调度的一个独立单位，是系统中的并发执行的单位。
- **线程**：线程是进程的一个实体，也是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位，有时又被称为轻量级进程。
- 进程是**资源分配**的最小单位，而线程是**CPU调度**的最小单位；
- 创建进程或撤销进程，系统都要为之分配或回收资源，操作系统开销远大于创建或撤销线程时的开销；
- 不同进程地址空间相互独立，同一进程内的线程共享同一地址空间。一个进程的线程在另一个进程内是不可见的；
- 进程间不会相互影响，而一个线程挂掉将可能导致整个进程挂掉；

**举个栗子**
计算机中的核心是CPU，说它是我们的大脑一点不为过。在此用一个连锁火锅店举例，因为疫情的影响，今天到目前营业额一般，只好开一家店，其他疫情较为严重的店只好先关闭，这里涉及的含义：单个CPU一次只运行一个任务。进程就类似这家火锅店，它代表CPU所能处理的单个任务，其他地方火锅店只能处于非运行状态。

一个火锅店有很多服务员，一起协同工作，将火锅店做的红红火火，那么线程就好比这些服务员，一个进程可有多个线程。

火锅店各个工作房间是共享的，所有服务员都可以进出，这意味着进程的内存空间共享，每个线程都可以使用这些共享内存。但是不是每个房间都可以容纳相同的人数，比如卫生间就只有一个人，其他人不能同时进去。这意味着一个线程在使用共享内存的时候，其他线程需要等待结束才能使用这块内存。那怎么防止别人进入呢？很直接的办法就是进去之后记得关门(上锁)，上了锁，其他人想进来发现是上锁了，那就等锁打开后再进去，这就叫做互斥锁，防止多个线程同时读写某一块内存区域。

**总结**
- 如果以多进程的方式运行，那么允许多个任务同时运行
- 如果以多线程的方式运行，那么允许将单个任务分成不同的部分运行
- 为了防止进程/线程之间产生冲突和允许进程之间的共享资源，需要提供一种协调机制。

### 多线程与多进程的基本概念
**举个栗子**
在学校食堂打菜，如果学校食堂就开设一个窗口，打菜的阿姨也没办法，只好一个个给大家依次打菜，这就好比"单线程",效率非常低。

为了提高效率，在食堂多加了几个窗口，这就类似"多线程"形式。

那么又想起一个问题，“多线程一定就比单线程效率高吗？”（ps Memcache是多线程模型而Redis是单线程模型)

### 貌似我们一提到高并发，分布式，就不得不想起多线程，那么多线程一定比单线程效率高？
上面说了采用多线程多核效果更好，但是多线程对CPU，内存等都要求比较高，如果存在的上下文切换过于耗时，互斥时间太久，效率反而会低。

不一定。不从专业术语来讲，举个例子，假设目前接水房有四个水管可以接水，我如果用4个桶分别对应4个水管，那么就比较完美，如果少一个则闲置一个，多一个则会出现抢占。如果此时我的水桶个数大于水管数，为了每个桶都有水，我们就需要切换水桶，这个过程实际上就是线程的上下文切换，代价一样不小。

### 多线程与多进程的应用场景
**多线程的优点**

- 更加高效的内存共享。多进程下内存共享不便
- 较轻的上下文切换。因为不用切换地址空间，CR3寄存器和清空TLB

**多进程的优点**

- 各个进程有自己内存空间，所以具有更强的容错性，不至于一个集成crash导致系统崩溃
- 具有更好的多核可伸缩性，因为进程将地址空间，页表等进行了隔离，在多核的系统上可伸缩性更强

### 如何提升多线程的效率

- 尽量使用池化技术，也就是线程池，从而不用频繁的创建，销毁线程
- 减少线程之间的同步和通信
- 通过Huge Page的方式避免产生大量的缺页异常
- 避免需要频繁共享写的数据

## 进程的状态转换
在Linux中，进程的状态有七种

### 可运行状态
英文名词为TASK_RUNNING，其实这个状态虽然是RUNING，实际上并不一定会占有CPU，可能修改TASK_RUNABLE会更妥当。TASK_RUNGING根据是否在在CPU上运行分为RUNGING和READY两种状态。处于READY状态的进程随时可以运行，只不过因为此时CPU资源受限，调度器没选中运行。

![](https://api2.mubu.com/v3/document_image/9c95bc60-be75-4252-9d5b-b6f9ef5bd7ae-3807603.jpg)

### 可中断睡眠状态与不可中断睡眠状态
我们知道进程不可能一直处于可运行的状态。假设A进程需要读取磁盘中的文件，这样的系统调用消耗时间较长，进程需要等待较长的时间才能执行后面的命令，而且等待的时间还是不可估算的，这样的话进程还占用CPU就不友好了，因此内核就会将其更改为其他的状态并从CPU可运行的队列移除。

Linux中存在两种睡眠状态，分别为：可中断的睡眠状态和不可中断的状态。两者最大的区别为是否响应收到的信号，那么从可中断的睡眠的进程是如何返回到可运行的状态呢?

- 等待的事情发生且继续运行的条件满足
- 收到了没有被屏蔽的信号

处于此状态的进程，收到信号会返回EINTR给用户空间。开发者通过检测返回值的方式进行后续逻辑处理。

但是对于不可中断的睡眠状态，就只有一种方式返回到可运行状态，即等待事情发生了继续运行。

![](https://api2.mubu.com/v3/document_image/3df04b2b-5036-4440-8378-f5a1f9d9449f-3807603.jpg)

上图中为什么出现个 TASK_UNINTERRUPTIBLE 状态，主要是因为内核中的进程不是什么进程都可以被打断，假设响应的是异步信号，程序在执行的过程中插入一段用于处理异步信号的流程，原来的流程就会被中断。所以当进程在和硬件打交道的时候，需要使用 TASK_UNINTERRUPTIBLE 状态将进程保护起来，从而避免进程和设备打交道的过程中被打断导致设备处于不可控的状态。

*那么TASK_UNINTERRUPTIBLE状态会出现多久呢？*

其实 TASK_UNINTERRUPTIBLE 状态是很危险的状态，因为它刀枪不入，你无法通过信号杀死这样一个不可中断的休眠状态，正常情况，TASK_UNINTERRUPTIBLE状态存在时间很短，但是不排除存在此状态进程比较持久的情况，真的刀枪不入了？可不可以进行提前的预防？

可以的，早就考虑了。内核提供了hung task机制，它会启动一个khungtaskd内核线程对TASK_UNINTERRUPTIBLE状态进行检测，不能让他失控了。khungtaskd会定期的唤醒，如果超过120s都还没有调度，内核就会通过打印警告和堆栈信息。当然，不一定就是120s，可以通过下面选项进行定制。

```linux
sysctl kernel.hung_task_timeout_secs
```

说了这么多，我们怎么知道到底有没有出现这个状态，哪里看？可以通过/proc和ps进行查看。

### 睡眠进程和等待队列
不管是上面提到的可中断的睡眠进程还是不可中断的睡眠进程，都离不开一种数据结构---队列。哦？假设进程A因为某某原因需要休眠，为啥要休眠，等待的资源迟迟拿不到或者等待的事件总是不来，没法进行下一步操作，这个时候内核来了，"行吧，我不会抛弃你，我一定会想办法让你和等待的资源(事件)扯上关系"，只要等待的时机到来我就唤醒你，这采用的方法即"等待队列"。

- TASK_STOPPED状态于TASK_TRACED状态

TASK_STOPPED状态属于比较特殊的状态，可以通过SIGCONT信号回复进程的执行

![](https://api2.mubu.com/v3/document_image/77ff07e5-42aa-44aa-9d66-14ea015361c3-3807603.jpg)

TASK_TRACED是被跟踪的状态，进程会停下来等待跟踪它的进程对它进行进一步的操作

### EXIT_ZOMBIE状态与EXIT_DEAD状态
当进程处于这两种的任意一种，就可以宣布"死亡" 。

![](https://api2.mubu.com/v3/document_image/62a9d653-309d-419f-951b-3719699c8599-3807603.jpg)

**就绪 —> 执行**：准备就绪，调度器满足了的需求，给我一种策略，我就可从就绪变为执行的状态；

**执行 —> 阻塞**：不是每个进程都是那么一帆风顺，就像我们每次考试，不管是中考，高考还是考研，难免都会出现磕磕跘跘，遇到了可能暂时会阻挡我们前行的小事儿，可是要相信不会一直的阻挡我们，只要我们有恒心坚持，时机到来，你也准备好了，那就美哉。回到这里，对于进程而言，当需要等到某个事情发生而无法执行的时候，进程就变为阻塞的状态。比如当前进程提出输入请求，如进程提出输入/输出请求，进程所申请资源（主存空间或外部设备）得不到满足时变成等待资源状态，进程运行中出现了故障（程序出错或主存储器读写错等）变成等待干预状态等等；

**阻塞 —> 就绪**：处于阻塞状态的进程，在其等待的事件已经发生，如输入/输出完成，资源得到满足或错误处理完毕时，处于等待状态的进程并不马上转入执行状态，而是先转入就绪状态，然后再由系统进程调度程序在适当的时候将该进程转为执行状态；

**执行 —> 就绪**：正在执行的进程，因时间片用完而被暂停执行，或在采用抢先式优先级调度算法的系统中,当有更高优先级的进程要运行而被迫让出处理机时，该进程便由执行状态转变为就绪状态。

## 进程间的通信方式有哪些？
### 管道
学习软件工程规范的时候，我们知道瀑布模型，在整个项目开发过程分为多个阶段，上一阶段的输出作为下一阶段的输入。各个阶段的具体内容如下图所示：

![](https://api2.mubu.com/v3/document_image/0bb11d60-c16b-4786-8667-1b86d5e41677-3807603.jpg)

最初我们在学习Linux基本命令使用的时候，我们经常通过多个命令的组合来完成我们的需求。比如说我们想知道如何查看进程或者端口是否在使用，会使用下面的这条命令

```linux
netstat -nlp | grep XXX
```

这里的"|"实际上就是管道的意思。"|"前面部分作为"|"后面的输入，很明显是单向的传输，这样的管道我们叫做"匿名管道"，自行创建和销毁。既然有匿名管道，应该就有带名字的管道"命名管道"。如果你想双向传输，可以考虑使用两个管道拼接即可。

**创建命名管道的方式**

```linux
mkfifo test
```

test即为管道的名称，在Linux中一切皆文件，管道也是以文件的方式存在，咋们可以使用ls -l 查看下文件的属性，它会"p"标识。

![](https://api2.mubu.com/v3/document_image/ce537067-cbd8-486b-b31d-5b7b5ef4ab33-3807603.jpg)

下面我们向管道写入内容

```linux
echo "666" > test
```

![](https://api2.mubu.com/v3/document_image/c504c1cb-30b5-41d5-acc6-0e9c53bb88e1-3807603.jpg)

此时按道理来说咱们已经将内容写入了test，没有直接输出是因为我们需要开启另一个终端进行输出(可以理解为暂存管道)。

```linux
cat < test
```

ok，我们发现管道内容被读出来，同时echo退出。那么管道这种通信方式有什么缺点？我们知道瀑布模型的软件开发模式是非常低下的，同理采用管道进行通信的效率也很低，因为假设现在有AB两个进程，A进程将数据写入管道，B进程需要等待A进程将信息写完以后才能读出来，所以这种方案不适合频繁的通信。那优点是什么？

最明显的优点就是简单，我们平时经常使用以致于都不知道这是管道。鉴于上面的缺点，我们怎么去弥补呢？接着往下看。

### 消息队列
管道通信属于一股脑的输入，能不能稍微温柔点有规矩点的发送消息？

答：可以的。消息队列在发送数据的时候，按照一个个独立单元(消息体)进行发送，其中每个消息体规定大小块，同时发送方和接收方约定好消息类型或者正文的格式。

在管道中，其大小受限且只能承载无格式字节流的方式，而消息队列允许不同进程以消息队列的形式发送给任意的进程。

但是当发送到消息队列的数据太大，需要拷贝的时间也就越多，所以还有其他的方式？继续看

### 共享内存
使用消息队列可以达到不错的效果，但是如果我们两个部门需要交换比较大的数据的时候，一发一收还是不能及时的感知数据。能不能更好的办法，双方能很快的分享内容数据，答：有的，共享内存

我们知道每个进程都有自己的虚拟内存空间，不同的进程映射到不同的物理内存空间。那么我们可不可以申请一块虚拟地址空间，不同进程通过这块虚拟地址空间映射到相同的物理地址空间呢？这样不同进程就可以及时的感知进程都干了啥，就不需要再拷贝来拷贝去。

我们可以通过shmget创建一份共享内存，并可以通过ipcs命令查看我们创建的共享内存。此时如果一个进程需要访问这段内存，需要将这个内存加载到自己虚拟地址空间的一个位置，让内核给它一个合法地址。使用完毕接触板顶并删除内存对象。

那么问题来了，这么多进程都共享这块内存，如果同时都往里面写内容，难免会出现冲突的现象，比如A进程写了数字5，B进程同样的地址写了6就直接给覆盖了，这样就不友好了，怎么办？继续往下看

### 信号量
为了防止冲突，我们得有个约束或者说一种保护机制。使得同一份共享的资源只能一个进程使用，这里就出现了信号量机制。

信号量实际上是一个计数器，这里需要注意下，信号量主要实现进程之间的同步和互斥，而不是存储通信内容。

信号量定义了两种操作，p操作和v操作，p操作为申请资源，会将数值减去M，表示这部分被他使用了，其他进程暂时不能用。v操作是归还资源操作，告知归还了资源可以用这部分。

### 信号
从管道————消息队列————共享内存/信号量，有需要等待的管道机制，共享内存空间的进程通信方式，还有一种特殊的方式————信号

我们或许听说过运维或者部分开发需要7 * 24小时值守(项目需要上线的时候)，当然也有各种监管，告警系统，一旦出现系统资源紧张等问题就会告知开发或运维人员，对应到操作系统中，这就是信号。

在操作系统中，不同信号用不同的值表示，每个信号设置相应的函数，一旦进程发送某一个信号给另一个进程，另一进程将执行相应的函数进行处理。也就是说把可能出现的异常等问题准备好，一旦信号产生就执行相应的逻辑即可。

### 套接字
上面的几种方式都是单机情况下多个进程的通信方式，如果我想和相隔几千里的人通信怎么办？

这就需要套接字socket了。其实这玩意随处可见，我们平时的聊天，我们天天请求浏览器给予的响应等，都是他。

### 小结

- 管道
- 消息队列
- 共享内存
- 信号量
- 信号
- 套接字

## 进程的调度算法有哪些？
调度算法是指：调度程序是内核的重要组成部分，决定着下一个要运行的进程。那么根据系统的资源分配策略所规定的资源分配算法。常用的调度算法有：先来先服务调度算法、时间片轮转调度法、短作业优先调度算法、最短剩余时间优先、高响应比优先调度算法、优先级调度算法等等。

### 先来先服务调度算法
先来先服务让我们想起了队列的先进先出特性，每一次的调度都从队列中选择最先进入队列的投入运行。

![](https://api2.mubu.com/v3/document_image/ea075cc5-1fda-4e7c-bce2-2a73ac612180-3807603.jpg)

### 时间片轮转调度算法
先来理解轮转，假设当前进程A、B、C、D，按照进程到达的时间排序，而且每个进行都有着同样大小的时间片。如果这个进程在当前的时间片运行结束，啥事儿没有，直接将进程从队列移除完事儿。如果进程在这个时间片跑完都没有结束，进程变为等待状态，放在进程尾部直到所有进程执行完毕。

为什么进程要切换，切换无外乎是时间片够用或者不够用。如果时间片够用，那么进程可以运行到结束，结束后删除启动新的时间片。如果时间片不够用，对不起，暂时只能完成一部分任务(变为等待状态)，过后再等待 CPU  的调度。网上开源的代码太多，怎么实现，大家可以参照加深影响。

### 短作业优先调度算法
短作业优先调度算法，从名称可以清晰的知道「短作业」意味着执行时间比较短，「优先」代表执行顺序。结合就是"短者吃香"。那么多短吃香？进程不可能都短，也有需要执行时间比较长的进程怎么办？一直等待，直到饿死？而且有些进程比较紧急，能够得到先执行？这些都是此算法所出现的问题，然后出现下面的一些算法

### 最短剩余时间优先调度算法
最短剩余时间是针对最短进程优先增加了抢占机制的版本。在这种情况下，进程调度总是选择预期剩余时间最短的进程。当一个进程加入到就绪队列时，他可能比当前运行的进程具有更短的剩余时间，因此只要新进程就绪，调度程序就能可能抢占当前正在运行的进程。像最短进程优先一样，调度程序正在执行选择函数是必须有关于处理时间的估计，并且存在长进程饥饿的危险。

### 高响应比优先调度算法
什么是高响应比，有响应之前应该会有请求，相当于是请求+响应+优先，算是一种综合的调度算法。也就是它结合了短作业优先，先来先服务以及长作业的一些特性。ok，那么这三种是如何体现出来的

首先来说短作业优先。等待时间我们假设相等，服务时间很短，这样的话短作业就会有更高的优先权。

再来看先来先服务。假设服务时间相同，先来的自然等待时间较长，优先级越高。

上面说长作业很可能因为等待时间过长，容易饿死。在这里不会，仿佛像医生的这个职业，工作越久资历越老，优先级越来越高，越来越吃香

### 优先级调度算法
优先级调度算法每次从后备作业队列中选择优先级最髙的一个或几个作业，将它们调入内存，分配必要的资源，创建进程并放入就绪队列。在进程调度中，优先级调度算法每次从就绪队列中选择优先级最高的进程，将处理机分配给它，使之投入运行。

## 什么是死锁？
死锁，顾名思义就是导致线程卡死的锁冲突，例如下面的这种情况

![](https://api2.mubu.com/v3/document_image/8cb1346d-14bf-4860-bae3-ade0cfd7ceff-3807603.jpg)

线程1已经成功拿到了互斥量1，正在申请互斥量2，而同时在另一个CPU上，线程2已经拿到了互斥量2，正在申请互斥量1。彼此占有对方正在申请的互斥量，结局就是谁也没办法拿到想要的互斥量，于是死锁就发生了。

![](https://api2.mubu.com/v3/document_image/37fa9a20-3c78-4f95-b505-b1bc36ab793b-3807603.jpg)

*稍微复杂一点的情况*

![](https://api2.mubu.com/v3/document_image/efb472d7-446e-4469-911f-b06210516f60-3807603.jpg)

存在多个互斥量的情况下，避免死锁最简单的方法就是总是按照一定的先后顺序申请这些互斥量。还是以刚才的例子为例，如果每个线程都按照先申请互斥量 1 ，再申请互斥量 2 的顺序执行，死锁就不会发生。有些互斥量有明显的层级关系，但是也有一些互斥量原本就没有特定的层级关系，不过没有关系，可以人为干预，让所有的线程必须遵循同样的顺序来申请互斥量。

## 产生死锁的原因？
由于系统中存在一些不可剥夺资源，而当两个或两个以上进程占有自身资源，并请求对方资源时，会导致每个进程都无法向前推进，这就是死锁。

**竞争资源**

例如：系统中只有一台打印机，可供进程 A 使用，假定 A 已占用了打印机，若 B 继续要求打印机打印将被阻塞。

系统中的资源可以分为两类：

1. 可剥夺资源：是指某进程在获得这类资源后，该资源可以再被其他进程或系统剥夺， CPU   和主存均属于可剥夺性资源；
2. 不可剥夺资源，当系统把这类资源分配给某进程后，再不能强行收回，只能在进程用完后自行释放，如磁带机、打印机等。

**进程推进顺序不当**

例如：进程 A 和 进程 B 互相等待对方的数据。

## 死锁产生的必要条件？
**互斥**
要求各个资源互斥，如果这些资源都是可以共享的，那么多个进程直接共享即可，不会存在等待的尴尬场景

**非抢占**
要求进程所占有的资源使用完后主动释放即可，其他的进程休想抢占这些资源。原因很简单，如果可以抢占，直接拿就好了，不会进入尴尬的等待场景

要求进程是在占有（holding）至少一个资源的前提下，请求（waiting）新的资源的。由于新的资源被其它进程占有，此时，发出请求的进程就会带着自己占有的资源进入阻塞状态。假设 P1，P2 分别都需要 R1，R2 资源，如果是下面这种方式：

```linux
P1:          P2:
request(R1)  request(R2)
request(R2)  request(R1)
```

如果 P1 请求到了 R1 资源之后，P2 请求到了 R2 资源，那么此后不管是哪个进程再次请求资源，都是在占有资源的前提下请求的，此时就会带着这个资源陷入阻塞状态。P1 和 P2 需要互相等待，发生了死锁。
换一种情况：

```linux
P1:          P2:
request(R1)  request(R1)
request(R2)  request(R2) 
```

如果 P1 请求到了 R1 资源，那么 P2 在请求 R1 的时候虽然也会阻塞，但是是在不占有资源的情况下阻塞的，不像之前那样占有 R2。所以，此时 P1 可以正常完成任务并释放 R1，P2 拿到 R1 之后再去执行任务。这种情况就不会发生死锁。

**循环等待**
要求存在一条进程资源的循环等待链，链中的每一个进程占有的资源同时被另一个进程所请求。

发生死锁时一定有循环等待（因为是死锁的必要条件），但是发生循环等待的时候不一定会发生死锁。这是因为，如果循环等待链中的 P1 和 链外的 P6 都占有某个进程 P2 请求的资源，那么 P2 完全可以选择不等待 P1 释放该资源，而是等待 P6 释放资源。这样就不会发生死锁了。

## 解决死锁的基本方法？
如果我们已经知道死锁形成的必要条件，逐一攻破即可。

**破坏互斥**
通过与锁完全不同的同步方式CAS，CAS提供原子性支持，实现各种无锁的数据结构，不仅可以避免互斥锁带来的开销也可避免死锁问题。

**破坏不抢占**
如果一个线程已经获取到了一些锁，那么在这个线程释放锁之前这些锁是不会被强制抢占的。但是为了防止死锁的发生，我们可以选择让线程在获取后续的锁失败时主动放弃自己已经持有的锁并在之后重试整个任务，这样其他等待这些锁的线程就可以继续执行了。这样就完美了吗？当然不

这种方式虽然可以在一定程度上避免死锁，但是如果多个相互存在竞争的线程不断的放弃重启放弃循环，就会出现活锁的问题，此时线程虽然没有因为锁冲突被卡死，但是仍然会因为阻塞时间太长处于重试当中。我们可以给任务重试部分增加随机延迟时间，降低任务冲突的概率。

**破坏环路等待**
在实践的过程中，采用破坏环路等待的方式非常常见，这种技术叫做"锁排序"。很好理解，我们假设现在有个数组A，采用单向访问的方式(从前往后)，依次访问并加锁，这样一来，线程只会向前单向等待锁释放，自然也就无法形成一个环路了。

说到这里，我想说死锁不仅仅出现在多线程编程领域，在数据库的访问也是非常的常见，比如我们需要更新数据库的几行数据，就得先获取这些数据的锁，然后通过排序的方式阻止数据层发生死锁。

这样就完美了？当然没有，那会出现什么问题？

这种方案也存在它的缺点，比如在大型系统当中，不同模块直接解耦和隔离得非常彻底，不同模块开发人员不清楚其细节，在这样的情况下就很难做到整个系统层面的全局锁排序了。在这种情况下，我们可以对方案进行扩充，例如Linux在内存映射代码就使用了一种锁分组排序的方式来解决这个问题。锁分组排序首先按模块将锁分为了不同的组，每个组之间定义了严格的加锁顺序，然后再在组内对具体的锁按规则进行排序，这样就保证了全局的加锁顺序一致。在Linux的对应的源码顶部，我们可以看到有非常详尽的注释定义了明确的锁排序规则。

这种解决方案如果规模过大的话即使可以实现也会非常的脆弱，只要有一个加锁操作没有遵守锁排序规则就有可能会引发死锁。不过在像微服务之类解耦比较充分的场景下，只要架构拆分合理，任务模块尽可能小且不会将加锁范围扩大到模块之外，那么锁排序将是一种非常实用和便捷的死锁阻止技术

## 怎么预防死锁？
- **破坏请求条件**：一次性分配所有资源，这样就不会再有请求了；
- **破坏请保持条件**：只要有一个资源得不到分配，也不给这个进程分配其他的资源：
- **破坏不可剥夺条件**：当某进程获得了部分资源，但得不到其它资源，则释放已占有的资源；
- **破坏环路等待条件**：系统给每类资源赋予一个编号，每一个进程按编号递增的顺序请求资源，释放则相反。

## 怎么避免死锁？
### 银行家算法
当进程首次申请资源时，要测试该进程对资源的最大需求量，如果系统现存的资源可以满足它的最大需求量则按当前的申请量分配资源，否则就推迟分配。

当进程在执行中继续申请资源时，先测试该进程已占用的资源数与本次申请资源数之和是否超过了该进程对资源的最大需求量。若超过则拒绝分配资源。若没超过则再测试系统现存的资源能否满足该进程尚需的最大资源量，若满足则按当前的申请量分配资源，否则也要推迟分配。

### 安全序列
是指系统能按某种进程推进顺序（P1, P2, P3, …, Pn），为每个进程 Pi 分配其所需要的资源，直至满足每个进程对资源的最大需求，使每个进程都可以顺序地完成。这种推进顺序就叫安全序列【银行家算法的核心就是找到一个安全序列】。

### 系统安全状态
如果系统能找到一个安全序列，就称系统处于安全状态，否则，就称系统处于不安全状态。

## 怎么解除死锁？
- **资源剥夺**：挂起某些死锁进程，并抢占它的资源，将这些资源分配给其他死锁进程（但应该防止被挂起的进程长时间得不到资源）；
- **撤销进程**：强制撤销部分、甚至全部死锁进程并剥夺这些进程的资源（撤销的原则可以按进程优先级和撤销进程代价的高低进行）；
- **进程回退**：让一个或多个进程回退到足以避免死锁的地步。进程回退时自愿释放资源而不是被剥夺。要求系统保持进程的历史信息，设置还原点。

## 什么是缓冲区溢出？有什么危害？
缓冲区溢出是指当计算机向缓冲区内填充数据时超过了缓冲区本身的容量，溢出的数据覆盖在合法数据上。

**举个栗子**
一个两升的杯子，你如果想倒入三升，怎么做？有一升只好流出去。

## 物理地址、逻辑地址、线性地址
- **物理地址**：它是地址转换的最终地址，是内存单元真正的地址。如果采用了分页机制，那么线性地址会通过页目录和页表的方式转换为物理地址。如果没有启用则线性地址即为物理地址。
- **逻辑地址**：在编写c语言的时候，通过&操作符可以读取指针变量本身的值，这个值就是逻辑地址。实际上是当前进程的数据段的地址，和真实的物理地址没有关系。只有当在Intel实模式下，逻辑地址==物理地址。我们平时的应用程序都是通过和逻辑地址打交道，至于分页，分段机制对他们而言是透明的。逻辑地址也称作虚拟地址。
- **线性地址**：线性地址是逻辑地址到物理地址的中间层。我们编写的代码会存在一个逻辑地址或者是段中的偏移地址，通过相应的计算(加上基地址)生成线性地址。此时如果采用了分页机制，那么吸纳行地址再经过变换即产生物理地址。在Intelk 80386中地址空间容量为4G，各个进程地址空间隔离，意味着每个进程独享4G线性空间。多个进程难免出现进程之间的切换，线性空间随之切换。基于分页机制，对于4GB的线性地址一部分会被映射到物理内存，一部分映射到磁盘作为交换文件，一部分没有映射。

## 分页与分段的区别？
我们知道计算机的五大组成部分分别为运算器，存储器，存储器 ，输入和输出设备。我们的数据或者指定都需要存放内存然后给 CPU  大哥拿去执行。我们平时写的代码不是直接操作的物理地址，我们所看到的地址实际上叫做虚拟地址，通过相应的转换规则将虚拟地址转换为物理地址。

**那么虚拟地址是怎么转换为物理地址的呢？**
第一种方式，采用一个映射表代表虚拟地址到物理地址的映射，在计算机中我们叫做页表。页表将内存地址分为页号和偏移量。

**举个栗子**
我们将高位部分称为内存地址的页号，后面的低位叫做内存地址的偏移量。我们只需要保存虚拟地址内存的页号和物理内存页号之间的映射关系即可。

![](https://api2.mubu.com/v3/document_image/7d81ffa1-55b6-444f-8184-031665897f45-3807603.jpg)

这样说了，也就是三部曲：

1. 虚拟地址-----> 页号+偏移量
2. 通过页表查询出虚拟页号，对应的物理页号
3. 物理页号+偏移量-----> 物理内存地址

![](https://api2.mubu.com/v3/document_image/58256929-657f-4863-b116-10bfc51215d5-3807603.jpg)

**这样的方法，在32位的内存地址，页表需要多大的空间？**
在一个32位的内存地址空间，页表需要记录2^20个物理页面的映射关系，可以想象为要给数组。那么一个页号是完整的4字节。这样一个页表就是4MB。

再来，我们知道进程有各自的虚拟内存空间，也就是说每个进程都需要一个这样的页表，不管此进程是只有几KB的程序还是需要GB的内存空间都需要这样的页表，用这样的结构保存页面，内存的占用将非常的大，那其他方式是怎么样的呢

**多级页表**
同样的虚拟内存地址，偏移量部分和上面方式一样，但是我们将页号部分拆分为四段，从高到低分成4级到1级的4个页表索引

![](https://api2.mubu.com/v3/document_image/84428943-a27b-4b1c-a158-f6dc12a159c9-3807603.jpg)

这样一来，每个进程将有4级页表。通过4级页表的索引找到对应的条目。通过这个条目找到3级页表所在位置，4级的每一个条目可能有多个3级的条目，找到了3级的条目后找到对应3级索引的条目，就这样到达1级页表。1级对应的则为物理页号。最终通过物理页号+偏移量的方式获取物理内存地址。

## 为什么使用多级页表
使用多级页表可以让页表在内存中离散存储。多级页表通过索引就可以定位到具体的项。举个例子，假设当前虚拟地址空间为4G，每个页的大小为4k，如果是一级页表的话，共有2……20个页表项，假设每个页表项需要4B，那么存放所有的页表项需要4M，那么为了随机访问，我们就需要连续的4M内存空间存放所有的页表项。这样一来，随着虚拟地址空间的增大，需要存放页表所需的连续空间也就越来多大。如果使用多级页表，我们只需要一页存放目录项，页表存放在内存其他位置即可，下面有例子进一步讲解

使用多级页表更加节省页表内存。理论上，使用一级页表，需要连续存储空间存放所有项。使用多级页表只需要给实际使用的的那些虚拟地址内存的请求分配内存

**举个栗子**
假设虚拟地址空间为4G，A进程只是用 4M 的内存空间。对于一级页表，我们需要 4M 空间存放这4GB 虚拟地址对应的页表，然后找到进程真正的 4M 内存空间。这样的话，A进程本来只使用 4MB 内存空间，但是为了访问它，我们需要为所有的虚拟地址空间建立页表，岂不是很浪费。对于二级页表而言，使用一个页目录就可定位 4M 的内存，存放一个页目录项需要 4k，还需要一页存放进程使用的 4M，4M=1024*4k，也就相当于 1024 个页表项就可以映射4M的内存空间，那么总共就只需要4k(页表)+4k(页目录)=8k来存放进程需要的 4M 内存空间对应页表和页目录项。这样看来确实剩下不少的内存。

**那使用多级页表有啥缺点？**
还是有的，使用一级页表的时候，只需要访问两次内存，一次是访问页表项，一次是访问需要读取的一页数据。如果是二级页表，就需要访问三次，第一次访问页目录，第二次访问页表项，第三次访问读取的数据。访问次数的增加以为访问数据所花费的总时间也增加

## 页面置换算法有哪些？
请求调页，也称按需调页，即对不在内存中的“页”，当进程执行时要用时才调入，否则有可能到程序结束时也不会调入。而内存中给页面留的位置是有限的，在内存中以帧为单位放置页面。为了防止请求调页的过程出现过多的内存页面错误（即需要的页面当前不在内存中，需要从硬盘中读数据，也即需要做页面的替换）而使得程序执行效率下降，我们需要设计一些页面置换算法，页面按照这些算法进行相互替换时，可以尽量达到较低的错误率。常用的页面置换算法如下：

**先进先出置换算法（FIFO）**
先进先出，即淘汰最早调入的页面。

**最佳置换算法（OPT）**
选未来最远将使用的页淘汰，是一种最优的方案，可以证明缺页数最小。

**最近最久未使用（LRU）算法**
即选择最近最久未使用的页面予以淘汰

**时钟（Clock）置换算法**
时钟置换算法也叫最近未用算法 NRU（Not RecentlyUsed）。该算法为每个页面设置一位访问位，将内存中的所有页面都通过链接指针链成一个循环队列。。

## 参考链接
[1.3w字，操作系统高频面试题大分享](https://mp.weixin.qq.com/s/oTEMOQY1xcG8uVceW-kLDA)
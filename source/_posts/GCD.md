---
title: GCD入门
date: 2016-09-20 20:26:49
categories: iOS
---

#### 前言

>  `GCD - Grand Central Dispatch`是Apple开发的一个多核编程的较新的解决方法，是苹果主推的多线程处理机制。在多核CPU的状态下，`GCD`的性能很高。
它自动利用更多的`CPU`内核，管理线程生命周期，程序员不需要编写任何线程管理代码，只需要给定要让`GCD`执行的任务。

<!--more-->

<br/>

> 2. `GCD`是纯C语言的，`GCD`中的函数大多数以`dispatch`开头。

> 3. `GCD`存在于 `libdispatch.dylib` 库中，但不需要手动导入，默认包含。

> 4. `GCD`的两个`核心概念`：`任务` 和 `队列`。将任务添加到`队列`中，`GCD`会自动将队列中的任务取出，放到对应的线程中执行，遵行FIFO原则：先进先出，后进后出。

> 5. `GCD`一般和`Block`一起使用，在`Block`回调中处理程序操作。

#### GCD的三种调度队列
> [官方文档传送门](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW1)

###### ** 首先需要明确`4`个术语的概念 **

* `线程`上的概念
	* 同步（sync）：函数要等到任务执行完才返回,所以会阻塞当前`线程`, 不是阻塞当前`队列`。

	* 异步（async）：函数马上返回, 异步向队列添加一个任务, 这个任务会被放到queue的队尾等待执行, 至于是串行还是并发执行和队列属性有关。

* `队列`上的概念
	* 并发（concurrent）：一段时间内多个任务(至于并发数到底是多少, 这个数量是在变化的, 与系统环境有关)都在同时执行。

	> 需要注意的是: 就算是`并行`队列, 任务的`开始时间`也是遵守`FIFO`的, 只是任务的执行不需要等待前一个任务完成后才开始而已。

	![image](https://cdn1.raywenderlich.com/wp-content/uploads/2014/01/Concurrent-Queue-480x272.png)

	* 串行（serial）：一个任务执行完成才执行下一个, 遵守FIFO。

	> 需要注意的是：你可以创建任意数量的串行队列, 并且这之中的每个队列相对于另外的队列并发执行。换句话说，如果你创建了4个串行队列，那么每个队列一次只执行一个任务，那么加起来就是4个任务在并发执行。

	![image](https://cdn2.raywenderlich.com/wp-content/uploads/2014/01/Serial-Queue-480x272.png)

##### 三种调度队列(dispatch queues)
* 运行在主线程的主队列的全局可获得的`main queue`，一般是执行和`UI`相关的任务比如更新`UI`的显示，通过`dispatch_get_main_queue`获取,不需要创建。
	```bash
    dispatch_queue_t dispatch_get_main_queue(void);
    ```
<br/>

* 并发队列`global dispatch queue`，一般是后台长时间执行的任务比如下载，可以通过`dispatch_get_global_queue`获取全局并行发队列或者通过(`iOS 5`及以后)`dispatch_queue_create`创建一个。

	```bash
	dispatch_queue_t dispatch_get_global_queue(dispatch_queue_priority_t priority,unsigned long flags)

	```

	```bash
	dispatch_queue_create(const char *label, dispatch_queue_attr_t attr)
    第一个参数是"队列名称"。第二个参数是"队列属性"，"并行发"队列传`"DISPATCH_QUEUE_GLOBAL"
	```

	```
	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	```

	```
	dispatch_queue_t queue = dispatch_queue_create("global_queue_name", DISPATCH_QUEUE_CONCURRENT);
	```
    
<br/>

* 串行队列`serial queues`，可以通过`dispatch_get_main_queue`获得主队列或者通过`dispatch_create`创建一个

	```bash
	dispatch_queue_t  dispatch_queue_create(const char *label,  dispatch_queue_attr_t attr)
    第一个参数是"队列名称"。第二个参数是"队列属性"，"串行"队列传"NULL/nil/DISPATCH_QUEUE_SERIAL"都是创建"串行"队列
	```
	```
	dispatch_queue_t queue = dispatch_queue_create("serial_queue_name",  NULL);
	```
<br/>
> Note: 系统同时提供给你好几个并发队列。它们叫做全局调度队列（Global Dispatch Queues）。目前的四个全局队列有着不同的优先级：background、low、default 以及 high。要知道，Apple的API也会使用这些队列，所以你添加的任何任务都不会是这些队列中唯一的任务。

	> From [Here](https://www.raywenderlich.com/60749/grand-central-dispatch-in-depth-part-1) and [译文](https://github.com/nixzhu/dev-blog/blob/master/2014-04-19-grand-central-dispatch-in-depth-part-1.md)

#### 关于`dispatch queues`的另外一些关键点

* dispatch queue之间，任务的执行是并发的。任务的串行执行是相对单个dislatch queue而言的.

* 系统决定了在同一时间段执行的任务数量。因此，有100个任务(每个都在一个队列里，所以有100个队列)的应用也许不会并发执行这100个任务，除非它有100或以上个有效的核。

* 当在选择要开始哪个队列的任务执行时，系统会考虑队列的优先级。

* 在一个队列里的任务在它们被添加到队列时，一定是做好了执行准备的。

* serial dispatch queue是引用计数对象。如果要在代码在retain它们，注意dispatch sources 能与一个队列联系同时也增加队列的引用计数。因此，你需要确保所有的dispatch sources都被取消，并且每个retain与每个release平衡对应。


> 附：`并行(parallesim)`与`并发(concurrency)`的区别
> [具体解释传送门](https://laike9m.com/blog/huan-zai-yi-huo-bing-fa-he-bing-xing,61/)
> 有一张图可以大概说明
![image](https://cdn3.raywenderlich.com/wp-content/uploads/2014/01/Concurrency_vs_Parallelism.png)

> 由于`concurrent`的翻译是`并发`，所以我没有用过`并行`这个词，至于用得对不对，我不清楚。


#### 线程死锁(deadlock)
##### 什么是死锁？

> ** `死锁`发生的四个必要条件 **
* `互斥`条件：一个资源每次只能被一个进程使用。
* `请求与保持`条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
* `不剥夺`条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
* `循环等待`(`形成回路`)条件:若干进程之间形成一种头尾相接的循环等待资源关系。

<br/>

说人话：有两个线程`A`和`B`都卡住了，因为`A`在等`B`，`B`也在等`A`，相互等待对方完成某些操作后自己才能进行操作，于是两者什么事也做不了，这就造成了`死锁`。

可以用`操作系统`中很经典的`图书馆问题`来说明:
图书馆里有100个座位，每个进入的人都需要登记后才能进去看书，离开前也需要登记后才能出去。

```bash
"V"表示信号量 + 1, "P"表示信号量 - 1
```

```
semaphore mutex = 1, seat = 100;
// 信号量初始化
P(seat); 检查有没有座位
P(mutex); 登记，登记行为的信号量 - 1
V(mutex); 登记完毕，释放登记信号量
/* 阅读... */
P(mutex); 要离开时需要登记
V(mutex); 释放登记信号量
V(seat); 座位信号量 + 1
```

这段伪代码没有问题，但是，当把`P(seat)`和`P(mutex)`交换一下

```bash
P(mutex);
P(seat);
```

想象当图书馆内已经`没有空闲座位`时，`P(mutex)`把登记资源占着，但是无法释放，因为`P(seat)`会因为`seat = 0`而陷入等待，而且占着座位的人想登记出去也登记不了，因为登记资源被占了，所以`seat`释放不了。
```bash
想进图书馆的"A"等着"seat"释放, 想出去的人"B"等着A释放"mutex"
```
<br/>
##### 什么样的代码会导致死锁？

```
dispatch_sync(diapatch_get_main_queue(),^{
	NSLog("线程死锁"); //任务１
})

```
###### 在程序中写这样的语句执行后不会有这句输出。
###### 很多地方的文章都写这个例子，然后告诉你只要不要`同步`往`main queue`里加任务就不会`死锁`，那么，这个说法太肤浅了，虽然我接下来写的东西也深入不到哪去，还没有研究过底层的源码。

[官方文档](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW1)里有这样一句话

> Submits a block to a dispatch queue for synchronous execution. Unlike dispatch_async, this function does not return until the block has finished. `Calling this function and targeting the current queue results in deadlock`.

按照官方文档里说的这句话，只要不要同步向当前队列添加任务，就不会导致死锁。









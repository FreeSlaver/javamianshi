---
layout: page
breadcrumb: true
title: Java多线程面试题最全
category: multi-thread-mians
categoryStr: 多线程面试题
tags: [multi-thread java 面试题]
keywords: [multi-thread java 面试题]
description: Java线程池面试题,JVM面试题大全
---




1. 多线程和单线程的区别和联系？
   1、 在单核 CPU 中，将 CPU 分为很小的时间片，在每一时刻只能有一个线程在执行，是一种微观上轮流占用 CPU 的机制。

2、 多线程会存在线程上下文切换，会导致程序执行速度变慢，即采用一个拥有两个线程的进程执行所需要的时间比一个线程的进程执行两次所需要的时间要多一些。

结论：即采用多线程不会提高程序的执行速度，反而会降低速度，但是对于用户来说，可以减少用户的响应时间。

2. 什么是多线程中的上下文切换？
   多线程会共同使用一组计算机上的CPU，而线程数大于给程序分配的CPU数量时，为了让各个线程都有执行的机会，就需要轮转使用CPU。不同的线程切换使用CPU发生的切换数据等就是上下文切换。

3. join方法的作用？
   Thread类中的join方法的主要作用就是同步，它可以使得线程之间的并行执行变为串行执行。当我们调用某个线程的这个方法时，这个方法会挂起调用线程，直到被调用线程结束执行，调用线程才会继续执行。

4. 创建线程池参数有哪些，作用？
   public ThreadPoolExecutor(   int corePoolSize,
   int maximumPoolSize,
   long keepAliveTime,
   TimeUnit unit,
   BlockingQueue<Runnable> workQueue,
   ThreadFactory threadFactory,
   RejectedExecutionHandler handler)
   1.corePoolSize:核心线程池大小，当提交一个任务时，线程池会创建一个线程来执行任务，即使其他空闲的核心线程能够执行新任务也会创建，等待需要执行的任务数大于线程核心大小就不会继续创建。

2.maximumPoolSize:线程池最大数，允许创建的最大线程数，如果队列满了，并且已经创建的线程数小于最大线程数，则会创建新的线程执行任务。如果是无界队列，这个参数基本没用。

3.keepAliveTime: 线程保持活动时间，线程池工作线程空闲后，保持存活的时间，所以如果任务很多，并且每个任务执行时间较短，可以调大时间，提高线程利用率。

4.unit: 线程保持活动时间单位，天（DAYS)、小时(HOURS)、分钟(MINUTES、毫秒MILLISECONDS)、微秒(MICROSECONDS)、纳秒(NANOSECONDS)

5.workQueue: 任务队列，保存等待执行的任务的阻塞队列。

一般来说可以选择如下阻塞队列：

ArrayBlockingQueue:基于数组的有界阻塞队列。

LinkedBlockingQueue:基于链表的阻塞队列。

SynchronizedQueue:一个不存储元素的阻塞队列。

PriorityBlockingQueue:一个具有优先级的阻塞队列。

6.threadFactory：设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。

7.handler: 饱和策略也叫拒绝策略。当队列和线程池都满了，即达到饱和状态。所以需要采取策略来处理新的任务。默认策略是AbortPolicy。

AbortPolicy:直接抛出异常。


CallerRunsPolicy: 调用者所在的线程来运行任务。


DiscardOldestPolicy:丢弃队列里最近的一个任务，并执行当前任务。


DiscardPolicy:不处理，直接丢掉。


当然可以根据自己的应用场景，实现RejectedExecutionHandler接口自定义策略。
5. 举例说明同步和异步。
   如果系统中存在临界资源（资源数量少于竞争资源的线程数量的资源），例如正在写的数据以后可能被另一个线程读到，或者正在读的数据可能已经被另一个线程写过了，那么这些数据就必须进行同步存取（数据库操作中的排他锁就是最好的例子）。当应用程序在对象上调用了一个需要花费很长时间来执行的方法，并且不希望让程序等待方法的返回时，就应该使用异步编程，在很多情况下采用异步途径往往更有效率。事实上，所谓的同步就是指阻塞式操作，而异步就是非阻塞式操作。

后面的问题，大家可以先自己独立思考一下。

另外我把所有Java相关的面试题和答案都整理出来了，给大家参考一下

面试题及答案PDF下载：https://www.hicxy.com/2645.html

面试题及答案PDF下载：https://www.hicxy.com/2645.html

面试题及答案PDF下载：https://www.hicxy.com/2645.html

6. 如何创建线程池
7. 为什么wait()方法和notify()/notifyAll()方法要在同步块中被调用
8. 为什么wait和notify方法要在同步块中调用？
9. join方法实现原理
10. notify()和notifyAll()有什么区别？
11. 为什么wait, notify 和 notifyAll这些方法不在thread类里面？
12. # 2、同步静态方法
13. 当一个线程进入某个对象的一个synchronized的实例方法后，其它线程是否可进入此对象的其它方法？
14. 你对线程优先级的理解是什么？
15. 关闭线程池
16. 线程同步和互斥有几种实现方法，都是什么？
17. 什么是Daemon线程？它有什么意义？
18. synchronized锁的是什么?
19. 在Java中CycliBarriar和CountdownLatch有什么区别？
20. wait 和 sleep 方法的不同?
21. 为什么Thread类的sleep()和yield ()方法是静态的？
22. Java中用到的线程调度算法是什么？
23. 现在有 T1、T2、T3 三个线程，你怎样保证 T2 在 T1 执行完后执行，T3 在 T2 执行完后执 行?
24. 在Java中Executor、ExecutorService、Executors的区别？
25. 乐观锁和悲观锁的理解及如何实现，有哪些实现方式？
26. 如何停止一个正在运行的线程
27. ThreadLocal是什么
28. 什么是Java内存模型
29. 什么是不可变对象，它对写并发应用有什么帮助？
30. 什么是原子操作？在Java Concurrency API中有哪些原子类(atomic classes)？
31. ConcurrentHashMap的并发度是什么
32. 说说自己是怎么使用 synchronized 关键字，在项目中用到了吗synchronized关键字最主要的三种使用方式：
33. Java中Semaphore是什么？
34. 如何在两个线程间共享数据？
35. 什么是ThreadLocal变量？
36. 怎么检测一个线程是否拥有锁？
37. 你如何在Java中获取线程堆栈？
38. 如何合理的设置线程池
39. Synchronized 有几种用法？
40. sleep方法和wait方法有什么区别
41. 什么是线程池？为什么要使用它？
42. 一个线程运行时发生异常会怎样？
43. 如何控制某个方法允许并发访问线程的大小？
44. 用户线程和守护线程有什么区别？
45. 线程中断是否能直接调用stop,为什么?
46. Runnable接口和Callable接口的区别
47. ThreadLocal有什么用
48. Java中堆和栈有什么不同？
49. Java中如何获取到线程dump文件
50. 什么是线程调度器(Thread Scheduler)和时间分片(Time Slicing)？
51. 不使用stop停止线程？
52. SynchronizedMap和ConcurrentHashMap有什么区别？
53. 高并发、任务执行时间短的业务怎样使用线程池？并发不高、任务执行时间长的业务怎样使用线程池？并发高、业务执行时间长的业务怎样使用线程池？
54. 你如何确保main()方法所在的线程是Java 程序最后结束的线程？
55. 同步方法和同步块，哪个是更好的选择？
56. 线程池作用
57. CopyOnWriteArrayList可以用于什么应用场景？
58. 如何创建守护线程？以及在什么场合来使用它？
59. 如何在两个线程之间共享数据
60. synchronized和ReentrantLock的区别
61. Thread.sleep(0)的作用是什么
62. 线程的sleep()方法和yield()方法有什么区别？
63. 什么是阻塞队列？阻塞队列的实现原理是什么？如何使用阻塞队列来实现生产者-消费者模型？
64. 在java中守护线程和本地线程区别？
65. 什么是Executors框架？
66. 什么是Java Timer 类？如何创建一个有特定时间间隔的任务？
67. 如何让正在运行的线程暂停一段时间？
68. 什么是自旋
69. Java中你怎样唤醒一个阻塞的线程？
70. Java中的死锁
71. start()方法和run()方法的区别
72. join与start调用顺序问题
73. Java Concurrency API中的Lock接口(Lock interface)是什么？对比同步它有什么优势？
74. 什么是线程池？ 为什么要使用它？
75. 线程的创建方式
76. 说一说自己对于 synchronized 关键字的了解
77. 为什么使用Executor框架？
78. 线程的状态
79. 简述线程、程序、进程的基本概念。以及他们之间关系是什么?
80. # 5、同步对象实例
81. # 1、同步普通方法
82. Executor框架的主要成员
83. synchronized包括哪两个jvm重要的指令？
84. Thread类中的yield方法有什么作用？
85. 线程类的构造方法、静态块是被哪个线程调用的
86. 什么是 FutureTask
87. volatile关键字的作用
88. 什么是线程安全？
89. volatile 变量和 atomic 变量有什么不同？
90. 为什么wait(), notify()和notifyAll ()必须在同步方法或者同步块中被调用？
91. 如何确保线程安全？
92. 什么是并发容器的实现？
93. # 3、同步类
94. 如何避免死锁和检测
95. 为什么使用Executor框架比使用应用创建和管理线程好？
96. wait()方法和notify()/notifyAll()方法在放弃对象监视器时有什么区别
97. 讲一下 synchronized 关键字的底层原理
98. Java中interrupted 和 isInterrupted方法的区别？
99. 什么是可重入锁（ReentrantLock）？
100. 线程池工作流程
101. Java中活锁和死锁有什么区别？
102. 线程的状态转换？
103.
104. 什么是 Callable 和 Future?
105. 线程有哪些基本状态?
106. 为什么我们调用start()方法时会执行run()方法，为什么我们不能直接调用run()方法？
107. 请说出与线程同步以及线程调度相关的方法。
108. 什么是阻塞（Blocking）和非阻塞（Non-Blocking）？
109. 什么是阻塞式方法？
110. Java中的同步集合与并发集合有什么区别？
111. 什么是竞态条件？你怎样发现和解决竞争？
112. 常用的线程池模式以及不同线程池的使用场景？
113. 死锁与活锁的区别，死锁与饥饿的区别？
114. Java中原子操作更新字段类，Atomic包提供了哪几个类?
115. 向线程池提交任务
116. join方法传参和不传参的区别？
117. # 4、同步this实例
118. 在线程中你怎么处理不可控制异常？
119. 在多线程中，什么是上下文切换(context-switching)？
120. 多线程有什么用？
121. 同步方法和同步块，哪个是更好的选择
122. 为什么代码会重排序？
123. Java中interrupted 和isInterruptedd方法的区别？
124. 什么是线程安全
125. 并行和并发有什么区别？
126. 自旋锁的优缺点？
127. Java线程池中submit() 和 execute()方法有什么区别？
128. 一个线程如果出现了运行时异常会怎么样
129. volatile 是什么?可以保证有序性吗?
130. ThreadLocal原理，使用注意点，应用场景有哪些？
131. 线程安全的级别
132. Java中notify 和 notifyAll有什么区别？
---
layout: page
breadcrumb: true
title: Java线程池面试题
category: multi-thread-mians
categoryStr: 多线程面试题
tags: [multi-thread java 面试题]
keywords: [multi-thread java 面试题]
description: Java线程池面试题,JVM面试题大全
---

## 1、什么是线程池
线程池的基本思想是一种对象池，在程序启动时就开辟一块内存空间，里面存放了众多(未死亡)的线程，池中线程执行调度由池管理器来处理。当有线程任务时，从池中取一个，执行完成后线程对象归池，这样可以避免反复创建线程对象所带来的性能开销，节省了系统的资源。

## 2、使用线程池的好处


减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
运用线程池能有效的控制线程最大并发数，可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。
对线程进行一些简单的管理，比如：延时执行、定时循环执行的策略等，运用线程池都能进行很好的实现


## 3、线程池的主要组件

一个线程池包括以下四个基本组成部分：

线程池管理器（ThreadPool）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
工作线程（WorkThread）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；
任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。

## 4、ThreadPoolExecutor类
讲到线程池，要重点介绍java.uitl.concurrent.ThreadPoolExecutor类，ThreadPoolExecutor线程池中最核心的一个类，ThreadPoolExecutor在JDK中线程池常用类UML类关系图如下：

我们可以通过ThreadPoolExecutor来创建一个线程池
new ThreadPoolExecutor(corePoolSize, maximumPoolSize,keepAliveTime,
milliseconds,runnableTaskQueue, threadFactory,handler);
复制代码
1. 创建一个线程池需要输入几个参数

corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。
ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常又帮助。
RejectedExecutionHandler（拒绝策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。n  AbortPolicy：直接抛出异常。
keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

2. 向线程池提交任务
   我们可以通过execute()或submit()两个方法向线程池提交任务，不过它们有所不同

execute()方法没有返回值，所以无法判断任务知否被线程池执行成功

threadsPool.execute(new Runnable() {
@Override
public void run() {
// TODO Auto-generated method stub
}
});
复制代码

submit()方法返回一个future,那么我们可以通过这个future来判断任务是否执行成功，通过future的get方法来获取返回值

try {
Object s = future.get();
} catch (InterruptedException e) {
// 处理中断异常
} catch (ExecutionException e) {
// 处理无法执行任务异常
} finally {
// 关闭线程池
executor.shutdown();
}
复制代码
3. 线程池的关闭
   我们可以通过shutdown()或shutdownNow()方法来关闭线程池，不过它们也有所不同

shutdown的原理是只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。
shutdownNow的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。shutdownNow会首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。

## 4. ThreadPoolExecutor执行的策略
   /**
    * Executes the given task sometime in the future.  The task
    * may execute in a new thread or in an existing pooled thread.
    *
    * If the task cannot be submitted for execution, either because this
    * executor has been shutdown or because its capacity has been reached,
    * the task is handled by the current {@code RejectedExecutionHandler}.
    *
    * @param command the task to execute
    * @throws RejectedExecutionException at discretion of
    *         {@code RejectedExecutionHandler}, if the task
    *         cannot be accepted for execution
    * @throws NullPointerException if {@code command} is null
      */
      public void execute(Runnable command) {
      if (command == null)
      throw new NullPointerException();
      /*
        * Proceed in 3 steps:
        *
        * 1. If fewer than corePoolSize threads are running, try to
        * start a new thread with the given command as its first
        * task.  The call to addWorker atomically checks runState and
        * workerCount, and so prevents false alarms that would add
        * threads when it shouldn't, by returning false.
        * 如果当前的线程数小于核心线程池的大小，根据现有的线程作为第一个Worker运行的线程，
        * 新建一个Worker，addWorker自动的检查当前线程池的状态和Worker的数量，
        * 防止线程池在不能添加线程的状态下添加线程
        *
        * 2. If a task can be successfully queued, then we still need
        * to double-check whether we should have added a thread
        * (because existing ones died since last checking) or that
        * the pool shut down since entry into this method. So we
        * recheck state and if necessary roll back the enqueuing if
        * stopped, or start a new thread if there are none.
        *  如果线程入队成功，然后还是要进行double-check的，因为线程池在入队之后状态是可能会发生变化的
        *
        * 3. If we cannot queue task, then we try to add a new
        * thread.  If it fails, we know we are shut down or saturated
        * and so reject the task.
        *
        * 如果task不能入队(队列满了)，这时候尝试增加一个新线程，如果增加失败那么当前的线程池状态变化了或者线程池已经满了
        * 然后拒绝task
          */
          int c = ctl.get();
          //当前的Worker的数量小于核心线程池大小时，新建一个Worker。
          if (workerCountOf(c) < corePoolSize) {
          if (addWorker(command, true))
          return;
          c = ctl.get();
          }
          //如果当前CorePool内的线程大于等于CorePoolSize，那么将线程加入到BlockingQueue。
          if (isRunning(c) && workQueue.offer(command)) {
          int recheck = ctl.get();
          if (! isRunning(recheck) && remove(command))//recheck防止线程池状态的突变，如果突变，那么将reject线程，防止workQueue中增加新线程
          reject(command);
          else if (workerCountOf(recheck) == 0)//上下两个操作都有addWorker的操作，但是如果在workQueue.offer的时候Worker变为0，
          //那么将没有Worker执行新的task，所以增加一个Worker.   addWorker(null, false);
          }
          //如果workQueue满了，那么这时候可能还没到线程池的maxnum，所以尝试增加一个Worker
          else if (!addWorker(command, false))
          reject(command);//如果Worker数量到达上限，那么就拒绝此线程
          }
          复制代码
## 5. 三种阻塞队列
   BlockingQueue workQueue = null;
   workQueue = new ArrayBlockingQueue<>(5);//基于数组的先进先出队列，有界
   workQueue = new LinkedBlockingQueue<>();//基于链表的先进先出队列，无界
   workQueue = new SynchronousQueue<>();//无缓冲的等待队列，无界


线程数量未达到corePoolSize，则新建一个线程(核心线程)执行任务
线程数量达到了corePools，则将任务移入队列等待
队列已满，新建线程(非核心线程)执行任务
队列已满，总线程数又达到了maximumPoolSize，就会由(RejectedExecutionHandler)抛出异常

新建线程  ->  达到核心数 -> 加入队列 -> 新建线程（非核心） -> 达到最大数 -> 触发拒绝策略
## 5. 四种拒绝策略

AbortPolicy：丢弃任务并抛出RejectedExecutionException异常

public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
throw new RejectedExecutionException("Task " + r.toString() +
" rejected from " +
e.toString());
}
复制代码

DiscardPolicy：丢弃任务，但是不抛出异常。

public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
}
复制代码

DisCardOldSetPolicy：丢弃队列最前面的任务，然后提交新来的任务

public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
if (!e.isShutdown()) {
e.getQueue().poll();
e.execute(r);
}
}
复制代码

CallerRunPolicy：由调用线程（提交任务的线程，主线程）处理该任务

public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
if (!e.isShutdown()) {
r.run();
}
}
复制代码
## 5、Java通过Executors提供5 种线程池
Executors 目前提供了 5 种不同的线程池创建配置：

newCachedThreadPool()，它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如果线程闲置的时间超过60 秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用 SynchronousQueue 作为工作队列。
newFixedThreadPool(int nThreads)，重用指定数目（nThreads）的线程，其背后使用的是无界的工作队列，任何时候最多有 nThreads 个工作线程是活动的。这意味着，如果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目 nThreads。
newSingleThreadExecutor()，它的特点在于工作线程数目被限制为 1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目。
newSingleThreadScheduledExecutor() 和 newScheduledThreadPool(int corePoolSize)，创建的是个 ScheduledExecutorService，可以进行定时或周期性的工作调度，区别在于单一工作线程还是多个工作线程。
newWorkStealingPool(int parallelism)，这是一个经常被人忽略的线程池，Java 8 才加入这个创建方法，其内部会构建ForkJoinPool，利用Work-Stealing算法，并行地处理任务，不保证处理顺序。

## 6、线程池参数设置
参数的设置跟系统的负载有直接的关系，下面为系统负载的相关参数：

tasks，每秒需要处理的的任务数（针对系统需求）
threadtasks，每个线程每钞可处理任务数（针对线程本身）
responsetime，系统允许任务最大的响应时间，比如每个任务的响应时间不得超过2秒。

corePoolSize
系统每秒有tasks个任务需要处理理，则每个线程每钞可处理threadtasks个任务。，则需要的线程数为：tasks/threadtasks，即tasks/threadtasks个线程数。
假设系统每秒任务数为100 ~ 1000，每个线程每钞可处理10个任务，则需要100 / 10至1000 / 10，即10 ~ 100个线程。那么corePoolSize应该设置为大于10，具体数字最好根据8020原则，因为系统每秒任务数为100 ~ 1000，即80%情况下系统每秒任务数小于1000 * 20% = 200，则corePoolSize可设置为200 / 10 = 20。
queueCapacity
任务队列的长度要根据核心线程数，以及系统对任务响应时间的要求有关。队列长度可以设置为 所有核心线程每秒处理任务数 * 每个任务响应时间 = 每秒任务总响应时间 ，即(corePoolSize*threadtasks)responsetime： (2010)*2=400，即队列长度可设置为400。
maxPoolSize
当系统负载达到最大值时，核心线程数已无法按时处理完所有任务，这时就需要增加线程。每秒200个任务需要20个线程，那么当每秒达到1000个任务时，则需要（tasks - queueCapacity）/ threadtasks 即(1000-400)/10，即60个线程，可将maxPoolSize设置为60。
队列长度设置过大，会导致任务响应时间过长，切忌以下写法：
LinkedBlockingQueue queue = new LinkedBlockingQueue();
复制代码
这实际上是将队列长度设置为Integer.MAX_VALUE，将会导致线程数量永远为corePoolSize，再也不会增加，当任务数量陡增时，任务响应时间也将随之陡增。
keepAliveTime
当负载降低时，可减少线程数量，当线程的空闲时间超过keepAliveTime，会自动释放线程资源。默认情况下线程池停止多余的线程并最少会保持corePoolSize个线程。
allowCoreThreadTimeout
默认情况下核心线程不会退出，可通过将该参数设置为true，让核心线程也退出。
一般说来，大家认为线程池的大小经验值应该这样设置：（其中N为CPU的个数）

如果是CPU密集型应用，则线程池大小设置为N+1
如果是IO密集型应用，则线程池大小设置为2N+1

## 7、线程池的五种状态


线程池的初始化状态是RUNNING，能够接收新任务，以及对已添加的任务进行处理。
线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。  调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。
线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。 调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。
当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。
当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

线程池彻底终止，就变成TERMINATED状态。线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。
## 8、关闭线程池
线程池提供两种关闭线程池方法：shutDown()和shutdownNow()
shutDown()
当线程池调用该方法时,线程池的状态则立刻变成SHUTDOWN状态。此时，则不能再往线程池中添加任何任务，否则将会抛出RejectedExecutionException异常。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。
shutdownNow()
根据JDK文档描述，大致意思是：执行该方法，线程池的状态立刻变成STOP状态，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，当然，它会返回那些未执行的任务。
它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，但是大家知道，这种方法的作用有限，如果线程中没有sleep、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。
## 9、各种场景下怎么设置线程数
### 1、高并发、任务执行时间短的业务怎样使用线程池？
线程池线程数可以设置为CPU核数+1，减少线程上下文的切换
### 2、并发不高、任务执行时间长的业务怎样使用线程池？
这个需要判断执行时间是耗在哪个地方


假如是业务时间长集中在IO操作上，也就是IO密集型的任务，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以适当加大线程池中的线程数目（2 * CPU核数），让CPU处理更多的业务。


假如是业务时间长集中在计算操作上，也就是CPU密集型任务，和（1）CPU核数+1 一样吧，线程池中的线程数设置得少一些，减少线程上下文的切换


###  3、并发高、业务执行时间长的业务怎样使用线程池？
解决这种类型任务的关键不在于线程池而在于整体架构的设计
## 10、为什么不推荐使用JUC的线程池？
这样的处理方式更加明确线程池的运行规则，规避资源耗尽的风险
1、newFixedThreadPool和newSingleThreadExecutor
上面两个主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM


2、newCachedThreadPool和newScheduledThreadPool
上面两个主要问题是最大线程数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

## 11、问题
### 1、非核心线程延迟死亡，如何实现？
通过阻塞队列poll()，让线程阻塞等待一段时间，如果没有取到任务，则线程死亡
### 2、线程池为什么能维持线程不释放，随时运行各种任务？
for (;;) {

            try {
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
复制代码
在死循环中工作队列workQueue会一直去拿任务:

核心线程的会一直卡在 workQueue.take()方法，让线程一直等待，直到获取到任务，然后返回。
非核心线程会 workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) ，如果超时还没有拿到，下一次循环判断compareAndDecrementWorkerCount就会返回null,Worker对象的run()方法循环体的判断为null,任务结束，然后线程被系统回收。

通过阻塞队列take()，让线程一直等待，直到获取到任务
### 3、如何释放核心线程？
将allowCoreThreadTimeOut设置为true。可用下面代码实验
{
// 允许释放核心线程，等待时间为100毫秒
es.allowCoreThreadTimeOut(true);
for(......){
// 向线程池里添加任务，任务内容为打印当前线程池线程数
Thread.currentThread().sleep(200);
}
}
复制代码
线程数会一直为1。 如果allowCoreThreadTimeOut为false，线程数会逐渐达到饱和，然后大家一起阻塞等待。
### 4、非核心线程能成为核心线程吗？
线程池不区分核心线程于非核心线程，只是根据当前线程池容量状态做不同的处理来进行调整，因此看起来像是有核心线程于非核心线程，实际上是满足线程池期望达到的并发状态。
### 5、Runnable在线程池里如何执行？
线程执行Worker，Worker不断从阻塞队列里获取任务来执行。。
1、newFixedThreadPool(固定数目的线程池);
使用场景：
fixedThreadPool核心线程池等于最大线程池，当前的线程数能够比较稳定保证一个数。能够避免频繁回收线程和创建线程。故适用于处理cpu密集型的任务，确保cpu在长期被工作线程使用的情况下，尽可能少的分配线程，即适用长期的任务。
此方法的弊端是：
到了线程池最大容量后，如果有任务完成让出占用线程，那么此线程就会一直处于等待状态，而不会消亡，直到下一个任务再次占用该线程。这就可能会使用无界队列来存放排队任务，当大量任务超过线程池最大容量需要处理时，队列无线增大，使服务器资源迅速耗尽。
2、newCachedThreadPool(可缓存线程的线程池);
使用场景：
newCacehedThreadPool 的最大特点就是，线程数量不固定。只要有空闲线程空闲时间超过keepAliveTime，就会被回收。有新的任务，查看是否有线程处于空闲状态，如果不是就直接创建新的任务。故适用用于并发不固定的短期小任务。
此方法的弊端是：
线程池没有最大线程数量限制，如果大量的任务同时提交，可能导致创线程过多会而导致资源耗尽。
线程池中多余的线程是如何回收的？
ThreadPoolExecutor回收工作线程，一条线程getTask()返回null，就会被回收。
分两种场景。

未调用shutdown() ，RUNNING状态下全部任务执行完成的场景

线程数量大于corePoolSize，线程超时阻塞，超时唤醒后CAS减少工作线程数，如果CAS成功，返回null，线程回收。否则进入下一次循环。当工作者线程数量小于等于corePoolSize，就可以一直阻塞了。

调用shutdown() ，全部任务执行完成的场景

shutdown() 会向所有线程发出中断信号，这时有两种可能。
2.1）所有线程都在阻塞
中断唤醒，进入循环，都符合第一个if判断条件，都返回null，所有线程回收。
2.2）任务还没有完全执行完
至少会有一条线程被回收。在processWorkerExit(Worker w, boolean completedAbruptly)方法里会调用tryTerminate()，向任意空闲线程发出中断信号。所有被阻塞的线程，最终都会被一个个唤醒，回收。


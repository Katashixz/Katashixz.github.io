---
title: 【Java知识点】关于并发的一些面试问答
date: 2023-03-13 14:40:51
categories: 自主学习
cover: ../image/Thread/cover.jpg
---

本篇文章内容参考[JavaGuide](https://javaguide.cn/)，自己整理一遍加深印象。

# 关于进程和线程

## 进程与线程的定义

- 进程
  - 程序的一次执行过程，是系统运行程序的基本单位，进程是动态的。
- 线程
  - 比进程更小的执行单位
  - 一个进程执行过程中可以产生多个线程
  - 同类的多个线程共享进程的**堆和方法区资源**
  - 每个线程有自己的**程序计数器、虚拟机栈和本地方法栈**
  - 也被称为轻量级进程

## 程序计数器为什么是私有的

一句话概括——为了让线程切换后能够恢复到正确的执行位置

##  堆和方法区的作用

- 是所有线程共享的资源
- 堆
  - 进程中最大的一块内存
  - 存放新创建的对象(几乎所有对象都是在这里分配内存)
- 方法区
  - 存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据

## 并发和并行的区别

- 并发：两个及两个以上的作业在同一**时间段**内执行
- 并行：两个及两个以上的作业在同一**时刻**执行

## 同步和异步的区别

- 同步： 发出一个调用之后，在没有得到结果之前，该调用就不可以返回，一直等待。
- 异步： 调用发出之后，不用等待返回结果，该调用直接返回。

## 线程的生命周期和状态

- NEW：初始状态，线程被创建出来，但是没有被调用start()
- RUNNABLE：运行状态，线程被调用了start()等待运行的状态
- BLOCKED：阻塞状态，需要等待锁释放
- WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作(通知或中断)
- TIME_WAITING：超时等待状态，可以再指定的时间后自行返回而不是像WAITING那样一直等待。
- TERMINATED：终止状态，表示该线程已经运行完毕。

线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。

![](../image//%E5%B9%B6%E5%8F%91%E5%B8%B8%E8%A7%81%E9%97%AE%E7%AD%94/lifeOfThread.png)

## 什么是上下文切换？

线程在执行过程中都会有自己的运行条件和状态(也称为上下文)，比如上文所说道过的程序计数器，栈信息等。当出现如下情况的时候，线程会从CPU状态中退出。
- 主动让出CPU，比如调用了`sleep()`,`wait()`等
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用CPU导致其他线程或者进程饿死
- 被调用了阻塞类型的系统中断，比如请求IO，线程被阻塞
- 被终止或结束运行

这其中前三种都会发生线程切换，发生线程切换意味着要保存当前线程的上下文，以便其下次占用CPU的时候恢复现场，并加载下一个将要占用CPU的线程上下文。这就是**上下文切换**。频繁的切换会造成整体效率低下，因为要占用CPU、内存等资源来保存和恢复信息。

## 什么是线程死锁？如何避免死锁？

### 定义

- 线程死锁描述的是一个场景。
- 多个线程同时被阻塞，他们互相等待彼此的资源释放，因此程序不可能被正常终止。
- 例
  - 线程A占有资源1，等待资源2，线程B占有资源2，等待资源1

### 产生死锁的四个必要条件

1. 互斥条件
   - 该资源任意时刻只有一个线程能占用
2. 请求与保持条件
   - 一个线程因请求资源而阻塞时，对已获得的资源保持不放
3. 不剥夺条件
   - 线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。 
4. 循环等待条件
   - 若干线程之间形成一种头尾相接的循环等待关系

### 如何预防死锁

只需要破坏死锁产生的必要条件即可
1. 破坏请求与保持条件
  - 一次性申请所有的资源
2. 破坏不剥夺条件
  - 占用部分资源的线程进一步申请其他资源时，如何申请不到，可以主动释放它占有的资源
3. 破坏循环等待条件
  - 靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放，破坏循环等待条件

### 如何避免死锁

避免死锁就是在资源分配时，借助于算法(例如银行家算法)对资源分配进行计算评估，使其进入**安全状态**
  > **安全状态**指系统能够按照某种线程推进顺序(P1,P2,P3...Pn)来为每个线程分配所需资源，直到满足每个线程对资源的最大需求，使每个线程都可以顺利完成。则称`<P1,P2,P3...Pn>`为安全序列。

## sleep()和wait()方法对比

- 共同点
  - 两者都可以暂停线程的执行
- 不同点
  - sleep()方法没有释放锁，而wait()方法释放了锁。
  - wait()方法通常被用于线程间交互/通信，sleep()通常被用于暂停执行。
  - wait()方法被调用后，线程不会自动苏醒，通常需要别的线程调用同一个对象上的notify()或者notifyAll()方法，sleep()方法执行完成后，线程会自动苏醒，或者也可以使用wait(long timeout)超时后线程会自动苏醒。
  - sleep()是Thread类的静态本地方法，wait()则是Object类的本地方法。

## 为什么wait()方法不定义在Thread中？

`wait()`是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。每个对象(`Object`)都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入WAITING状态，自然是要操作对应的对象(`Object`)而非当前的线程(`Thread`)。

## 可以直接调用Thread类的run方法吗？（常问）

一句话概括——调用`start()`方法可以启动线程并使线程进入就绪状态，直接执行`run()`方法的话不会以多线程的方式执行。

详解——new 一个`Thread`，线程进入了新建状态。调用`start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。`start()`会执行线程的相应准备工作，然后自动执行`run()`方法的内容，这是真正的多线程工作。 但是，直接执行`run()`方法，会把`run()`方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

# Java中的并发应对设计

## volatile关键字

在Java中，`volatile`关键字可以保证变量的**可见性**，用`volatile`关键字声明的变量是**共享且不稳定**的，每次使用它都到主存中进行读取。

`volatile`能保证数据的可见性，但是**不能保证数据的原子性**，`synchronized`两者都能保证。

在Java中，`volatile`关键字除了可以保证变量的可见性，还有一个重要的作用就是**防止JVM的指令重排序**。当我们将某一个变量声明为`volatile`，在对这个变量进行读写操作的时候，会通过插入特定的**内存屏障**的方式来禁止指令重排序。

### 双重检验锁方式实现单例模式的原理

放在设计模式知识点里复习好了。。怎么这么多(怒)

## 如何保证一个+1操作的原子性

1. 使用`synchronized`改进
  ```java
  public static int inc = 0;
  public synchronized void increase(){
    inc ++;
  }
  ```
2. 使用`AtomicInteger`改进
  ```java
  public AtomicInteger inc = new AtomicInteger();
  public void increase(){
    inc.getAndIncrement();
  }
  ```
3. 使用`ReentrantLock`改进
  ```java
  Lock lock = new ReentrantLock();
  public void increase(){
    lock.lock();
    try{
      inc ++;
    } finally{
      lock.unlock();
    }
  }
  ```

## 乐观锁和悲观锁

### 乐观锁

- 定义
  - 总是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停地执行，无需加锁也无需等待，只是在提交的时候去验证对应的资源(也就是数据)是否被其他线程修改了。
- 使用场景
  - 通常用于**读多写少**的场景，避免频繁加锁影响性能，大大提升了系统的吞吐量。
- 具体实现方法
  - 具体方法可以使用**版本号机制**或者**CAS算法(使用更多)**

### 悲观锁

- 定义
  - 总是假设最坏的情况，认为共享资源每次被访问的时候就会出现问题(比如共享资源被修改)，所以每次在获取资源操作的时候都会上锁。这样其他线程想拿到这个资源就会阻塞直到锁被上一个持有者释放。
- 使用场景
  - 通常用于**写多读少**的场景，避免频繁失败和重试影响性能。
- 具体实现方法
  - `synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

## 乐观锁的实现

### 版本号机制

一般是在数据表中加上一个数据版本号`version`，表示数据被修改的次数，当数据被修改时`version`值会加一。当线程A要更新数据值时，在读取数据的同时也会读取`version`，在提交时检查，如果刚才读取的`version`和当前数据库中的`version`相等，则更新，不相等会一直重试直到更新成功。

### CAS算法(CompareAnd Swap)

- 维护的三个操作数
  - V 将要更新的变量值(Var)
  - E 预期值(Expected)
  - N 拟写入的新值(New)
- 核心思想
  - 当且仅当V上的值和预期值E相等时，才会用新值N通过原子方式更新V，否则不执行更新。

## 乐观锁存在的问题

### ABA问题

如果一个线程在初次读取时的值为A，并且在准备赋值的时候检查该值仍然是A，但是可能在这两次操作之间，有另外一个线程现将变量的值改成了B，然后又将该值改回为A，那么CAS会误认为该变量没有变化过。

### 循环时间长、CPU开销大

CAS如果失败就会一直进行尝试，即一直在自旋，长时间不成功会给CPU带来非常大的开销。

### 只能保证一个共享变量的原子操作

CAS只对**单个共享变量有效**。操作涉及多个共享变量时CAS无效。多个变量可以利用锁或者AtomicReference类把多个共享变量合并成一个共享变量来操作。

## Synchronized 关键字

### 定义

`synchronized`是Java中的一个关键字，直译为同步。主要解决**多个线程之间访问资源的同步性**，可以保证被他修饰的方法或者代码块在**任意时刻只能有一个线程执行**。

### 如何使用synchronized

1. 修饰实例方法
   - 锁当前**对象实例**，进入同步代码前要获得**当前对象实例的锁** 
2. 修饰静态方法
   - 锁当前**类**，作用于类的**所有对象实例**，进入同步代码前要获得**当前class的锁**。这是因为静态成员不属于任何一个实例对象，归整个类所有，被类的所有实例共享。
3. 修饰代码块
   - 对**括号里的指定对象/类加锁**
   - synchronized(object) 表示进入同步代码库前要获得**给定对象的锁**
   - synchronized(类.class) 表示进入同步代码前要获得**给定Class的锁**

### 总结

1. `synchronized`关键字加到`static`静态方法和`synchronize(xx.class)`代码块上都是给类上锁
2. `synchronized`关键字加到实例方法上是给对象实例上锁
3. 尽量不要使用`synchronized(String a)`因为JVM中，字符串常量池具有缓存功能
4. 构造方法不能用`synchronized`修饰，本身就属于线程安全，不存在同步的构造方法一说。

### synchronized 和 volatile 有什么区别

`synchronized`和`volatile`是**互补**的存在，而不是对立的存在。
1. `volatile`是线程同步的轻量级实现，所以性能肯定比`synchronized`要好。
2. `synchronized`可以保证数据的原子性和可见性，`volatile`只能保证数据的可见性。
3. `synchronized`用于修饰静态方法、实例方法或代码块，`volatile`用于修饰变量
4. `volatile`主要用于解决**变量在多个线程之间的可见性**，而`synchronized`用于解决**多个线程之间访问资源的同步性**。

## ReentrantLock

### 定义

`ReetrantLock`是一个实现了`Lock`接口的可重入且独占式的锁，和`synchronized`关键字类似，不过更加灵活、强大，增加了**轮询、超时、中断、公平锁和非公平锁**等高级功能。

添加锁和释放锁的大部分操作实际上都是在`ReetrantLock`里的一个内部类`Sync`中实现的。`Sync`有公平锁`FairSync`和非公平锁`NofairSync`两个子类。`ReetrantLock`默认使用非公平锁，也可以通过构造器指定使用公平锁。

![](../image/Thread/reentrantlock-class-diagram.png)

### 公平锁和非公平锁的区别

- 公平锁
  - 锁被释放之后，**先申请的线程先得到锁**。性能较差一些，因为公平锁为了保证时间上的绝对顺序，上下文切换更频繁。

- 非公平锁
  - 锁被释放之后，**后申请的线程可能会先获取到锁**，是**随机**或者**按照其他优先级排序的**。性能更好，但可能导致某些线程永远无法获取到锁。

## synchronized 和 ReentrantLock 有什么区别

1. `synchronized`依赖于JVM而`ReentrantLock`依赖于API
   - `synchronized`依赖于JVM实现，并没有直接暴露给我们
   - `ReentrantLock`在JDK层面实现，需要lock()、unlock()方法配合try/finally语句块来完成，可以查看源代码。
2. `ReentrantLock`比`synchronized`增加了一些高级功能
   1. **等待可中断**。`ReentrantLock`提供了一种能够中断等待锁的线程和机制，通过`lock.lockInterruptibly()`来实现，也就是说正在等待的线程可以选择放弃等待，去处理其他事情。
   2. **可实现公平锁**
    - `ReentrantLock`可以指定公平锁还是非公平锁。默认是非公平的，通过构造方法来指定。
    - `synchronized`只能是非公平锁。
   3. **可实现选择性通知(锁可以绑定多个条件)**
    - `synchronized`关键字与`wait()`和`notify()`/`notifyAll()`方法相结合可以实现等待/通知机制。
    - `ReentrantLock`类需要借助于`Condition`接口与`newCondition`方法，使用上十分灵活，可以实现多路通知功能(一个`Lock`对象中可以创建多个`Condition`实例，即对象监视器)。**线程对象可以注册在指定的`Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活**。
  
## 可中断锁和不可中断锁有什么区别

- 可中断锁
  - 获取锁的过程中可以被中断，不需要一直等到获取锁之后，才能进行其他逻辑处理。`ReentrantLock`属于可中断锁。
- 不可中断锁
  - 一旦线程申请了锁，就只能等拿到锁以后才能进行其他的逻辑处理。`synchronized`属于不可中断锁。

# 线程池相关

## ThreadLocal

### 是什么

`ThreadLocal`类主要解决的是让每个线程都绑定自己的值，可以理解为学校储物柜，每个柜子放着私有的东西。

### 有什么用

如果创建了一个`ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的本地副本。可以使用`get()`方法来获取默认值，`set()`方法将其值改为当前线程所存的副本值，从而避免了线程安全问题。

### 原理

最终的变量是放在了当前线程的`ThreadLocalMap`中，并不是存在`ThreadLocal`上，`ThreadLocal`可以理解为只是`ThreadLocalMap`的封装,传递了变量值。`ThreadLocal`类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

### 可能会导致内存泄漏问题

`ThreadLocalMap`中使用的key为`ThreadLocal`的弱引用，而value是强引用，所以如果`ThreadLocal`没有被外部强引用的情况下，在垃圾回收的时候，key就会被清理掉，而value不会被清理掉。

这样情况就导致了出现key为null的entry，假如我们不做任何措施，value就永远无法被GC回收，就可能会产生内存泄漏。

`ThreadLocalMap`已经考虑了这种情况，所以在调用`set()`、`get()`、`remove()`方法的时候，会清理掉key为null的记录。最好自己手动调用`remove()`方法。

## 线程池

### 是什么

管理一系列线程的资源池，有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。

### 为什么要用

池化技术主要就是为了**减少每次获取资源的消耗，提高对资源的利用率**。线程池提供了一种限制和管理资源的方式，每个线程池也会维护一些基本统计信息，例如已完成的任务数量。

### 优势

摘自《Java并发编程的艺术》
  > **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
  > **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
  > **提高线程的可管理性**。线程是有限的资源，如果无限制的创建不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配、调优和监控。

### 如何创建线程池

1. 通过`ThreadPoolExecutor`构造函数来创建(推荐)
![](https://javaguide.cn/assets/threadpoolexecutor%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0.d54a5992.png)
   ```java
   /**
     * ThreadPoolExecutor 类中提供的四个构造方法。我们来研究最长的那个
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
    public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                              int maximumPoolSize,//线程池的最大线程数
                              long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                               )

   ```

1. 通过`Executor`框架的工具类`Executors`创建



### 为什么不推荐使用内置线程池

使用线程池的好处就是**减少在创建和销毁线程上锁消耗的时间以及系统资源的开销**，解决资源不足的问题。如果不使用线程池，有可能会造成系统创建大量同类线程而导致消耗完内存或者"过度切换"的问题

在《阿里巴巴Java开发手册》中，明确指出线程资源必须通过线程池提供，**不允许在应用中自行显式创建线程**。且强制线程池**不允许使用`Executors`去创建**，而是通过`ThreadPoolExecutor`构造函数的方式，这样的处理方式让写的同学更加明确线程池的运行规则，避免资源耗尽的风险。

### 使用`Executors`返回线程池对象的弊端

1. `FixedThreadPool`和`SingleThreadExecutor`使用的是无界的`LinkedBlockingQueue`，任务队列最大长度为`Integer.MAX_VALUE`，可以堆积大量请求而导致OOM。
2. `CachedThreadPool`使用的是同步队列`SynchronousQueue`，允许创建的线程数量为`Integer.MAX_VALUE`，可能会创建大量线程从而导致OOM。
3. `ScheduledThreadPool`和`SingleThreadScheduledExecutor`使用的无界的延迟阻塞队列`DelayedWorkQueue`，任务队列最大长度为`Integer.MAX_VALUE`，可能堆积大量请求，从而导致OOM。

### `ThreadPoolExecutor`的一些参数

**最重要的参数**
1. `corePoolSize`
   - 任务队列未达到队列容量时，最大可以同时运行的线程数量
2. `maximumPoolSize`
   - 任务队列达到队列容量，并且小于`maximumPoolSize`的时候，会创建新线程来执行任务，当前可以同时运行的线程数量转变为`maximumPoolSize`指定的值。
3. `workQueue`
  - 新任务来的时候会先判断当前运行的线程数量是否到达`corePoolSize`，到达就会被存放在这里。

其他常见参数
1. `unit`
   - `keepAliveTime`参数的时间单位
2. `handler`
   - 饱和策略。
3. `threadFactory`
   - executor创建新线程的时候会用到。
4. `keepAliveTime`
   - 线程池中的线程数量大于`corePoolSize`的时候，如果这时没有新任务提交，核心线程外的线程不会立即销毁，而是等待，直到等待时间超过设定值才会被销毁。
  
图片来源《Java性能调优实战》

![](https://javaguide.cn/assets/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%90%84%E4%B8%AA%E5%8F%82%E6%95%B0%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB.d65f3309.png)

### 线程池的饱和策略

如果**当前同时运行的线程数量达到最大线程并且队列也已经被放满了任务**时，`ThreadPoolTaskExecutor`定义了一些策略

1. `ThreadPoolExecutor.AbortPolicy`
   - 抛出`RejectedExecutionException`来拒绝新任务的处理
2. `ThreadPoolExecutor.CallerRunsPolicy`
   - 用调用者所在的线程来执行任务，也就是直接在调用`execute`方法的线程中运行(`主线程中`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务的提交速度，影响程序的整体性能。在特定业务场景(每一个任务都要求被执行)下可以选择。
3. `ThreadPoolExecutor.DiscardPolicy`
   - 不处理新任务直接丢掉
4. `ThreadPoolExecutor.DiscardOldestPolicy`
   - 此策略将丢弃最早的未处理的任务请求

### 线程池处理任务的流程

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/javaguide/%E5%9B%BE%E8%A7%A3%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.png)

1. 首先，任务在提交之后，会先检测核心线程池是否已满，未满则创建线程去处理任务。
2. 如果已满，则会检测等待队列是否已满，未满则加入队列。
3. 如果已满，会检查线程池是否已满(检查是否线程数是否到达`maximumPoolSize`)，未满则创建线程。
4. 如果已满，则会按照饱和策略处理。

### 如何给线程池命名

初始化线程池的时候需要显示命名(设置线程池名称前缀)，有利于定位问题。通常使用以下两种方式。

1. 利用guava的`ThreadFactoryBuilder`
   ```java
   ThreadFactory threadFactory = new ThreadFactoryBuilder()
                           .setNameFormat(threadNamePrefix + "-%d")
                           .setDaemon(true).build();
   ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit.MINUTES, workQueue, threadFactory)
   ```
2. 自己实现`ThreadFactor`
   ```java
   import java.util.concurrent.Executors;
   import java.util.concurrent.ThreadFactory;
   import java.util.concurrent.atomic.AtomicInteger;
   /**
    * 线程工厂，它设置线程名称，有利于我们定位问题。
    */
   public final class NamingThreadFactory implements ThreadFactory {

       private final AtomicInteger threadNum = new AtomicInteger();
       private final ThreadFactory delegate;
       private final String name;

       /**
        * 创建一个带名字的线程池生产工厂
        */
       public NamingThreadFactory(ThreadFactory delegate, String name) {
           this.delegate = delegate;
           this.name = name; // TODO consider uniquifying this
       }

       @Override
       public Thread newThread(Runnable r) {
           Thread t = delegate.newThread(r);
           t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
           return t;
       }
   }
   ```

### 如何设定线程池的大小

线程池的大小并不是越大越好，对于多线程场景来说要考虑上下文切换的成本。
  > 上下文切换是**任务从保存到在加载的过程**。多线程编程中一般线程的个数都大于CPU核心的个数，而一个CPU核心在任意时刻只能被一个线程使用。为了线程都能有效被执行，CPU采取的策略是为每个线程**分配时间片并轮转**的形式。当一个线程的时间片用完的时候就会重新出于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。概括来说就是——当前任务在执行完CPU时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以在加载这个任务的状态。 
  > 上下文切换通常是**密集型**的，需要相当可观的处理器时间。所以上下文切换对系统来说意味着**会消耗大量的CPU时间**。

以下有一个简单并且适用面比较广的公式。
  - CPU密集型任务(N+1)
    - 利用CPU计算能力的任务，例如大量数据排序。
    - 这种任务消耗的主要是CPU资源，可以将线程数设置为N(CPU核心数) + 1。比CPU核心数多出来的一个线程是为了**防止线程偶发的缺页中断**，或者其他原因导致的任务暂停而带来的影响。一旦任务暂停，也可以充分利用CPU，不让他停下。
  - I/O密集型任务(2N)
    - 大部分时间花在等待IO操作完成上的任务，例如文件读取。
    - 这类任务系统会用大部分时间来处理I/O交互，在这段时间CPU是空闲的，所以可以交给其他线程使用。因此可以多配置一些线程。

### Future类

是异步思想的典型运用，主要用在一些需要执行耗时任务的场景，避免程序一直原地等待耗时任务执行完成。即把耗时任务交给子线程去异步执行，主线程做其他事，最后通过`Future`类去获取耗时任务的执行结果。

Java中的`Future`类只是一个泛型接口，其中定义了五个方法，主要包括四个功能
1. 取消任务
2. 判断任务是否被取消
3. 判断任务是否已经执行完成
4. 获取任务执行结果

### AQS

AQS的全称为`AbstractQueuedSynchronizer`，抽象队列同步器，是一个抽象类，主要用来**构件锁和同步器**。

AQS为构建锁和同步器提供了一些通用功能的实现，因此，使用AQS能简单且高效地构造出应用广泛的大量的同步器。比如`ReentrantLock`、`ReentrantReadWriteLock`、`SynchronousQueue`等等皆是基于AQS的。

### AQS的原理

核心思想
- 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用**CLH队列锁**实现的，即将暂时获取不到锁的线程加入到队列中。
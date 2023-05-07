## Java并发(https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html)

### JAVA内存模型(JMM,Java Memoery Model)

CPU使用了2个优化方案提高了运算性能，同时这两个方案引出了一个新的并发问题：一致性问题

1. 在多核处理器的每个核上使用了高速缓存，并同步回主内存，这引起了缓存一致性问题
2. 为了充分利用运算单元，对输入代码进行优化，这一优化可能会改变运算的执行顺序，虽然可以保证乱序后的运算执行结果和乱序前一致，但是如果另一个CPU的运算依赖该运算的中间结果，则可能影响到另一个运算的执行结果，这导致了多线程的安全性问题

<img src="https://pic3.zhimg.com/80/v2-a1a75c9f7264cf78d0927663371ca9d2_1440w.jpg" alt="img" style="zoom: 25%;" />



Java虚拟机规范中定义了Java内存模型（Java Memory Model，JMM），用于屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的并发效果

Java内存模型采用了类似的方案，且引入了统一的交互协议解决并发问题

<img src="https://pic3.zhimg.com/80/v2-037270b0876b6af680d1832bcc9dca32_1440w.jpg" alt="img" style="zoom: 50%;" />

交互协议

- **lock（锁定）**：作用于主内存的变量，把一个变量标识为一条线程独占状态。
- **unlock（解锁）**：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- **read（读取）**：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- **load（载入）**：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- **use（使用）**：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- **assign（赋值）**：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- **store（存储）**：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- **write（写入）**：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

---

#### 进程与线程

```
- 进程是程序的一个实例，进程是资源分配的最小单位(网络IO，内存，磁盘)
- 进程内可以分为1到多个线程，一个线程是一串指令流，线程是CPU时间片的最小调度单位
- 进程通信需要依赖协议，线程通信只需要基于共享进程内的共享变量
- 线程的上下文切换比进程要低很多
```

#### 并行与并发

```
- 并行是同一时间同时处理多件事情
- 并发是同一时间应对多件事情的能力(单核CPU下，基于分配CPU时间片的方案串行处理多件事情)
- 应用
  - 单核CPU下，多线程不能提高程序运行效率，但是可以解决某一子任务阻塞的情况下，切换子任务去有效利用CPU资源
  - IO操作不占用CPU，只是我们一般拷贝文件使用的阻塞IO，这时由于阻塞会导致CPU要等待IO结束，没能充分利用资源；所以非阻塞IO和异步IO因此诞生了(异步IO方案优于非阻塞IO)
```

### 线程

#### 创建线程

```java
//方法1
Thread t = new Thread("t"){
  public void run(){
    
  }
};
//方法2
Runable runable = new Runable(){
  public void run{}
};
Thread t1 = new Thread(runable,"t1");
//方法3
FutureTask<Integer>task = new FutureTask<>(() -> {
  log.debug("hello");
  return 100;
})
new Thread(task3, "t3").start();
//此处线程阻塞等待线程task执行完毕返回结果
Integer result = task3.get();
```

#### 栈与栈帧

JVM由栈、堆、方法区组成，栈内存主要用于线程的执行

```
- 栈解决线程的执行问题(比如方法的返参，入参)
- 堆解决数据的存储问题(栈内对象的创建只有引用，实际数据会存储在堆里面)
- 每个栈内每个方法的调用会产生一个栈帧(Frame), 同时存储方法调用时所占用的内存(IDEA debug可见)
- 每个线程只能有一个active栈帧，对应正在执行的方法
```

#### 线程上下文切换

```
线程在以下原因下会不再继续执行，转而执行另一个线程的代码，这种线程我们称之为线程的上下文切换
	- 分配给线程的CPU时间片用完了
	- 垃圾回收
	- 有更高优先级的线程需要运动(垃圾回收就属于更要优先级)
	- 线程自己调用sleep, yield, wait, join, park, synchronized, lock等方法(线程挂起，线程阻塞)
当上下文切换的时候，操作系统需要保存当前线程的状态，并回复另一条线程的状态
	- 当前线程的状态-》程序计数器，栈帧信息，局部变量，操作数栈，返回地址等
	- ***频繁的上下文切换会影响到性能***
```

| 方法名           | 功能说明                              | 备注                                                         |
| ---------------- | ------------------------------------- | ------------------------------------------------------------ |
| join             | 等待某个线程结束                      |                                                              |
| getId            | 获取线程长整形ID(唯一)                |                                                              |
| getName          | 获取线程名                            |                                                              |
| setPriority(int) | 设置线程优先级                        | JAVA中线程优先级为1-10                                       |
| getState         | 获取线程状态                          | NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED        |
| isInterrupted    | 是否被打断                            | 线程被打断会产生```打断标记```                               |
| interrupt        | 打断线程                              | interrupt打断sleep，join, wait状态的线程会清除打断标记，打断运行中的线程，线程不会受到影响，只会做打断标记 |
| sleep(long)      | 休眠,static                           | 休眠当前正在执行的线程，单位毫秒                             |
| yield            | 提示线程调度器让出当前线程的CPU时间片 | 具体现象依赖操作系统的任务调度器                             |

#### 线程优先级

- 调度器在调度线程时不会强依赖线程优先级
- 如果CPU比较忙，那么高优先级的线程会获得更多的时间片，但CPU闲时，优先级几乎没用

#### 不推荐的线程方法（容易造成线程死锁）

| 方法名  | 功能说明     | 备注 |
| ------- | ------------ | ---- |
| stop    | 停止线程运行 |      |
| suspend | 挂起线程     |      |
| resume  | 恢复线程运行 |      |

#### 线程安全性分析

```
- 成员变量和静态变量是否线程安全
  - 如果没有共享，则线程安全
  - 如果已经被分享
    - 如果只有读操作，则线程安全
    - 如果有读写操作，则这段代码是临界区，需要考虑线程安全
- 局部变量是否线程安全
  - 局部变量是线程安全的
  - 但局部变量引用的对象不一定
    - 如果引用的是局部对象则线程安全
    - 如果引用的是共享变量，则需要考虑线程安全
```



-----

#### synchronized

```java
synchronized本质上等价于对象锁，根据使用方式不同，分为锁定实例和类本身
  - 非静态方法，synchronized会锁定类实例
	- 静态方法，synchronized会锁定类本身
```

在JVM中，对象存储包含了3部分信息（对象头，成员变量，补位数据），补位数据主要用于8个字节的对齐

以32位虚拟机为例

<img src="https://gitee.com/gu_chun_bo/picture/raw/master/image/20200308224345-655905.png" alt="1583678624634" style="zoom:80%;" />

普通对象结构

|-------------------------------------------------------------------------|

|							<font color="red">Object Header（64 bits）	</font>			 |

|-------------------------------------------------------------------------|

|	<font color="green">Mark Word（32 bits）</font> |	<font color="green">Klass Word（32 bits）</font>|

|-------------------------------------------------------------------------|



数组对象结构

|----------------------------------------------------------------------------------------------------------------|

|													<font color="red">Object Header（96 bits）	</font>									 |

|----------------------------------------------------------------------------------------------------------------|

|	<font color="green">Mark Word（32 bits）</font> |	<font color="green">Klass Word（32 bits）</font>|	<font color="green">Array Length（32 bits）</font>|

|----------------------------------------------------------------------------------------------------------------|



Mark Word结构如下

![1583651590160](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200308151311-525787.png)

对象头包含了Mark Word，当对象被执行到synchronized的时候Mark Word会关联到管程Monitor，并用一个2位数的状态码标记当前状态，Monitor则会记录Owner为当前线程，新线程在竞争锁的时候则会进入Monitor的EntryList，直到Owner被释放为空



Mark Word会根据锁的使用情况标记锁的竞争

![img](https://pic3.zhimg.com/80/v2-e935f5f175cbf34ceeb54647be22b9e6_1440w.jpg)





JDK1.6 对 Synchronized 锁这块的优化，在之前只有无锁与重量级锁两种，因此我们以前会说 Synchronized 较重，性能较差；但是 JDK1.6 对其优化之后，在非极端情况下，Synchronized 是很难『膨胀』到重量级锁状态的。 



- 锁的状态：无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁；



```
1. 偏向锁

大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得，那么此时锁就是偏向锁。
如果一个线程获得了锁，那么锁就进入偏向模式，此时 Mark Word 记录该线程的 ID；当该线程再次请求锁时，无需做任何同步操作，即需要在获取锁的时候检查一下 Mark Word 的锁标记位 = 偏向锁，并且 threadID = 该线程ID 即可，因此，省去了锁申请的操作。

2. 轻量级锁

当有第二个线程来申请该对象锁时，偏向锁则立即升级为轻量级锁。(在当前栈帧中创建一个LockRecord，用于存储Mark Word的拷贝，并且将Mark Word指向该LockRecord)

3. 重量级锁

同理，当同一时间有多个线程竞争锁时，锁就会立即升级为重量级锁，此时申请锁带来的开锁也就变大。
```

---

#### Volatile

Volatile本质上保证的是原子变量的可见性，正常由于线程内存区缓存了变量，会导致变量的值发生改变存在不可见性，而volatile保证线程总是从主存中获取变量的值



- 通过插入内存屏障指令禁止编译器和CPU对程序进行重排序。

- 当对声明了volatile的变量进行写操作时，JVM就会向处理器发送一条Lock前缀的指令，这条Lock前缀指令产生如下两个作用：
  - Lock前缀指令会引起处理器缓存回写到系统内存，并使用缓存一致性机制来确保回写的原子性。
  - 一个处理器的缓存回写到系统内存会导致其他处理器的缓存无效。处理器使用MESI控制协议去维护内部缓存和其他处理器缓存的一致性。处理器能嗅探其他处理器访问系统内存和它们的内部缓存。处理器使用嗅探技术保证它的内部缓存、系统内存和其他处理器的缓存的数据在总线上保持一致。例如，在Pentium和P6 family处理器中，如果通过嗅探一个处理器来检测其他处理器打算写内存地址，而这个地址当前处于共享状态，那么正在嗅探的处理器将使它的缓存行无效，在下次访问相同内存地址时，强制执行缓存行填充。

### CAS

- CAS操作的就是乐观锁，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。

- 1.  **ABA问题**。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。

  **从Java1**.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

  关于ABA问题参考文档: http://blog.hesey.net/2011/09/resolve-aba-by-atomicstampedreference.html

  **2. 循环时间长开销大**。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

   

  **3. 只能保证一个共享变量的原子操作**。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了**AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。**(其实就是UNSAFE.compareAndSwapObject的应用)



### AtomicStampedReference

AtomicStampedReference用于解决CAS产生ABA问题的场景

1. 定义了Pair<value,Stamp>分别存储值和版本号
2. 基于CAS，如果当前value=传入的value，且当前stamp=传入的stamp，则可以更新为新的value和新的stamp
3. 为了保证原子性，本质是调用UNSAFE.compareAndSwapObject(this, pairOffset, myPair, newPair)



## J.U.C(java.util.concurrent)

### AQS(AbstractQueuedSynchronizer)

AQS是阻塞锁和相关同步器工具的框架

```
1. AQS的底层数据结构与Monitor的实现方案类似,
2. AQS是一个抽象类，提供了volatile status用于标记锁状态，由子类实现tryAcquire并调用内部的CAS方法实现status加锁和解锁的状态值
3. 使用AQS需要Override部分方法：tryAcquire,tryRelease,isHeldExclusively(是否被当前线程独占),tryAcquireShared,tryReleaseShared
3. AQS同时提供了一个双向链表作为队列，用于存储由于锁失败被阻塞的线程，并使用LockSupport.park和unpark实现线程的阻塞和唤醒逻辑
4. AQS会记录锁持有的线程是哪个，子类可以基于这一信息实现锁的可重入
```

### ReentrantLock(可重入锁)

相比较synchronized

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

```
try{
	reentrantLock.lock();
	//临界区
} finally {
	//释放锁
	reentrantLock.unlock();
}
```

ReentrantLock的实现基于队列同步器（AbstractQueuedSynchronizer，后面简称AQS）

ReentrantLock的可重入功能基于AQS的同步状态：state。

其原理大致为：当某一线程获取锁后，将state值+1，并记录下当前持有锁的线程，再有线程来获取锁时，判断这个线程与持有锁的线程是否是同一个线程，如果是，将state值再+1，如果不是，阻塞线程。 当线程释放锁时，将state值-1，当state值减为0时，表示当前线程彻底释放了锁，然后将记录当前持有锁的线程的那个字段设置为null，并唤醒其他线程，使其重新竞争锁。



---





#### 线程池

- ExecutorService
  - ScheduledExecutorService
    - ScheduledThreadPoolExecutor
  - ThreadPoolExecutor
    - ScheduledThreadPoolExecutor



```
/**
     * @param corePoolSize 核心线程数(最多保留的线程数)
     * @param maximumPoolSize 最大线程数
     * @param keepAliveTime 生存时间，针对救急线程
     * @param unit 时间单位，针对救急线程，与keepAliveTime配合使用
     * @param workQueue 阻塞队列
     * @param threadFactory 线程工厂，用于定义线程创建的逻辑，比如分配线程名
     * @param handler 拒绝策略
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) 
```

- RejectedExecutionHandler拒绝策略
  - DiscardPolicy：抛出RejectedExecutionException异常，默认策略
  - CallerRunsPolicy让调用者运行任务
  - AbortPolicy放弃本次任务
  - DiscardOldestPolicy放弃队列中最早的任务，用本任务代替



- Executors
  - newFixedThreadPool（固定池，核心线程数=最大线程数）
  - newSingleThreadExecutor（单线程池，核心线程数=最大线程数=1）
  - newCachedThreadPool（缓存池，核心线程为0，最大线程数=无限大，采用SynchronousQueue，只要请求过来就会丢给线程，如果没有空闲线程，则会创建新的空闲线程，线程的存活时间为60S）
  - newScheduledThreadPool（延时队列，任务延时执行）
  - newWorkStealingPool（拥有多个队列的线程池，基于ForkJoinPool实现）



注意点：超过核心线程数，新的任务会进入阻塞队列，当阻塞队列满的时候，此时会创建新的线程用于消化，直到线程数达到最大线程数，最大线程数=核心线程数+救急线程数。另外队列的创建很重要，如果队列容量不作考虑，则永远不会走到救急线程，且存在OOM的风险。






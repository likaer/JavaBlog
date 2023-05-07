## JAVA基础知识

#### Java中有几种数据类型

```
整形：byte,short,int,long
浮点型：float,double
字符型：char
布尔型：boolean
```

#### Object类常用方法有那些？

```Equals
Hashcode//计算hash值
toString//转字符串
wait//触发线程等待
notify//唤醒等待的线程
clone//对象克隆
getClass//获取类名
finalize//在对象回收前调用，覆写该方法可以对系统资源等进行回收
```

#### Final在java中的作用

```Final可以修饰类，修饰方法，修饰变量。
修饰的类叫最终类。该类不能被继承。
修饰的方法不能被重写。
修饰的变量叫常量，常量必须初始化，一旦初始化后，常量的值不能发生改变。
```

#### String, StringBuffer，Stringbuilder有什么区别？

```
String是不可变类, StringBuffer与StringBuilder都继承了AbstractStringBulder类，而AbtractStringBuilder又实现了CharSequence接口，两个类都是用来进行字符串操作的。
StringBuffer基于synchronized关键字保证了线程安全，Stringbuilder是非线程安全的；因此StringBuilder效率上会高于StringBuffer
```

#### Java中异常有哪些

```
1. Java中所有异常都源自于Throwable
2. Error和Exception继承自Throwable，Error通常指系统级别的异常，如内存溢出，堆栈溢出等等。
3. Exception分为checkedExpcetion和uncheckedExpcetion，也就是必须捕获处理的异常和运行时异常，前者包含IO异常等，后者包含RuntimeException，数组越界异常等
```

#### HashMap的原理(线程不安全)

```
DEFAULT_INITIAL_CAPACITY Table数组的初始化长度： 1 << 42^4=16（为什么要是 2的n次方？）
MAXIMUM_CAPACITY Table数组的最大长度： 1<<302^30=1073741824
DEFAULT_LOAD_FACTOR 负载因子：默认值为0.75。 当元素的总个数>当前数组的长度 * 负载因子。数组会进行扩容，扩容为原来的两倍（todo：为什么是两倍？）
TREEIFY_THRESHOLD 链表树化阙值： 默认值为 8 。表示在一个node（Table）节点下的值的个数大于8时候，会将链表转换成为红黑树。
UNTREEIFY_THRESHOLD 红黑树链化阙值： 默认值为 6 。 表示在进行扩容期间，单个Node节点下的红黑树节点的个数小于6时候，会将红黑树转化成为链表。
MIN_TREEIFY_CAPACITY = 64 最小树化阈值，当Table所有元素超过改值，才会进行树化（为了防止前期阶段频繁扩容和树化过程冲突）。


1. 对Key做计算HashCode，然后根据数组长度取余映射映射到数组Hash表，对映射重合的数据用链表存储
2. 在JDK1.8中当链表的长度大于8会采用红黑树去存储代替链表
3. Table数组的初始化长度为16, 当元素的总个数>当前数组的长度 * 负载因子(默认75%)。数组会进行扩容，扩容为原来的两倍
4. 扩容=将老table数组中所有的Entry取出来，重新对其Hashcode做Hash散列到新的Table中
5. hash算法Arrays.hashCode(){result = 31 * result + (element == null ? 0 : element.hashCode());}
```

#### ConcurrentHashMap(JDK 1.8)

JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现**，**并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap

```
1. 对Key做计算HashCode，然后根据数组长度取余映射映射到数组Hash表，对映射重合的数据用链表存储
2. 当链表的长度大于8会采用红黑树去存储代替链表
3. 使用sizeCtl字段初始化和扩容hash表，使用CAS+volatile保证并发
4. 采用CAS+synchronized实现并发插入或更新操作
TODO 跟JDK1.6比较差异在哪里
1. 扩容阶段使用了CAS的方式保证了并发性，在扩容阶段通过设置sizeCtl字段为-1来保证并发
2. 由于数据结构采用Node存储，所以在插入和更新阶段，使用synchronized可以最小粒度锁定节点，保证最大并发
```

### IO与NIO

```
1. IO属于阻塞模型，当线程发生IO请求后，会等待IO执行完毕，但磁盘的操作是不消耗CPU的，因此这里面会存在资源的浪费，于是NIO诞生了
```





# <font color="blue">软件设计准则</font>

软件设计有两个终极目标：**高内聚、低耦合**

围绕这两个终极目标，提出了如何达到终极目标的六大设计细则

这些细则包括单一职责、开闭原则、里氏替换、依赖倒置、接口隔离、最少知识

#### 单一职责原则（Single Responsibility Principle, SRP）

**每个类应该实现单一的职责，否则就应该做拆分。**

```
职责的最细粒度很多时候会显得有点多余，但是 在未来的某些时候会发现往往在后续的迭代中，细粒度的职责可以使得软件的可扩展性大大提升，并且迭代的效率也会大幅度提升。
```

#### 开闭原则(Open-Closed Principle, OCP)

**软件中的对象（类，模块，函数等等）对于扩展是开放的，但是对于修改是封闭的**

```
为了满足开闭原则，需要对系统进行抽象化设计，抽象化是开闭原则的关键；JAVA中主要通过接口和抽象类对实现系统的抽象层，从而达到在不修改原有代码实现扩展的功能
```

#### 里式替换原则(Liskov Substitution Principle, LSP)

**任何基类可以出现的地方，衍生类一定可以出现**

```
里式替换是对开闭原则的补充，开闭原则定义了如何抽象化，而里式替换定义了如何实现抽象化
只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为
```

#### 接口隔离原则(Interface Segregation Principle, ISP)：

接口中不应该存在子类用不到却必须实现的方法，否则，就应当将接口拆分

```使用多个隔离的接口进行实现，比使用单个臃肿的接口要好。```

#### 依赖倒置原则(Dependency Inversion Principle, DIP)

1. 高层次的模块不应该依赖于低层次的模块，两者都应该依赖于[抽象接口](https://zh.wikipedia.org/wiki/抽象化_(計算機科學))

2. 抽象接口不应该依赖于具体实现。而具体实现则应该依赖于抽象接口

#### 最少知道原则(Law of Demeter, LoD)：

**一个类应该对其他类知道的越少越好**

```只有这样，当被依赖的类发生变化时，才能最少的影响到当前类。```

## JVM(JDK 1.8)

JVM被分为三个主要的子系统：类加载器子系统、运行时数据区和执行引擎 。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120710435597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NTQzNTA4,size_16,color_FFFFFF,t_70)

### JVM内存模型

![JVM内存模型](https://img.javatt.com/5b/5bc42dbda1a7b32f021c92dbca5465c4.jpeg)



```
### 程序计数器

当前线程所执行的字节码的行号指示器，用于记录正在执行的虚拟机字节指令地址，线程私有。
由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式实现，因此为了线程切换后能恢复到正确的位置，每条线程都需要一个独立的程序计数器

### 方法区

存储类信息、常量、静态变量、即时编译器编译后的代码，线程共享。JDK1.8采用元数据区定义方法区

### 堆（Heap）

线程共享。所有的对象实例以及数组都要在堆上分配。回收器主要管理的对象。

### 虚拟机栈（JVM Stack）

每个线程内的方法在执行的同时都会创建一个栈帧(Stack Frame)，用于存储局部变量表、操作栈、动态链接、方法出口，对象指针。

### 本地方法栈（Native Method Stack）

为虚拟机使用到的Native 方法服务。如Java使用c或者c++编写的接口服务时，代码在此区运行。
有的虚拟机会把本地方法栈和虚拟机栈合二为一
```

### Java堆

Java 堆是内存空间占据的最大一块区域了，Java 堆是用来存放**对象实例**及**数组**, 一个JVM只会有一个堆。

Java堆可以分为**新生代**和**老年代**，新生代还被进一步划分为 Eden 区、From Survivor 0、To Survivor 1 区

```
Java堆 = 老年代 + 新生代
新生代 = Eden + From Survivor + To Survivor
默认Eden：from ：to = 8:1:1
```

- Eden Space

字面意思是伊甸园，对象被创建的时候首先放到这个区域，进行垃圾回收后，不能被回收的对象被放入到空的survivor区域。

- Survivor Space幸存者区(2个Survivor区可以实现复制算法，每执行一次复制算法，两者角色发生互换)

用于保存在eden space内存区域中经过垃圾回收后没有被回收的对象。Survivor有两个，分别为To Survivor、 From Survivor，这个两个区域的空间大小是一样的。执行垃圾回收的时候Eden区域不能被回收的对象被放入到空的survivor（也就是To Survivor，同时Eden区域的内存会在垃圾回收的过程中全部释放），另一个survivor（即From Survivor）里不能被回收的对象也会被放入这个survivor（即To Survivor），然后To Survivor 和 From Survivor的标记会互换，始终保证一个survivor是空的。

Eden Space和Survivor Space都属于新生代，新生代中执行的垃圾回收被称之为Minor GC（因为是对新生代进行垃圾回收，所以又被称为Young GC），每一次Young GC后留下来的对象age加1。

- 老年代

1. YoungGC的对象没经过一次，age会加一，默认对象年龄=15的时候，会晋升到老年代，参数-XX:MaxTenuringThreshold
2. 大对象直接进入老年代
3. Survivor相同年龄的对象大小>=Surivor空间的一半的时候，这部分对象直接进入老年代(否则无法执行复制算法)

### 垃圾回收算法

----

可回收对象标记算法

```
### 引用计数算法
描述：在对象中添加一个引用计数器，初始值为0，每新增一个引用+1，每减少一个引用-1；
问题：存在互相循环依赖的问题，如objA.instance = objB; objB.instance.A, 除此以外两者再无其他引用，则此时这两个对象已经不可被访问，但引用计数器都为1，于是无法被回收

PS: 基本不采用该算法

### 可达性分析算法
描述：从GC ROOT出发，遍历所有的引用，如果不可达，则该对象为可回收对象；GC ROOT=虚拟机栈(栈帧)引用的对象+方法区引用的对象

### 标记清除算法
描述：算法分为标记和清除两个阶段，首先标记出需要回收的对象，在标记完成后统一回收所有被标记的对象
问题：效率存在问题+产生不连续的内存碎片

### 复制算法
描述：将内存分为等份的2份，每次只使用其中的一块，每当这一快内存快使用完了，就将还存活的对象复制到另外一块内存，再把这一块内存清理掉；
		该方法适合新生代内存回收，由于新生内对象朝生夕死，将新生代内存分为大的Eden和两块小的Survivor，其中两块Survivor用一块作为复制内存，另一块+Eden作为清理内存
问题：方法简单，运行高效，代价是内存缩小了一半


### 标记整理算法
描述：整合复制和标记算法的新思路，采用标记+清除+存活对象整理到一块



### 新生代和老年代由于性质不同，采用不同的算法可以提高收集的效率
1. 新生代由于每次收集大量对象死亡，选用复制算法只保留少部分存活对象就可以完成收集
2. 老年代对象存活率高，没有额外的空间进行担保，所以应该用”标记清除“或者“标记整理”算法
```



### 垃圾收集器(垃圾回收算法的具体实现)

### 垃圾回收

JDK 8 默认打开了`UseParallelGC`参数，因此使用了`Parallel Scavenge + Serial Old`的收集器组合进行内存回收

---

#### Parallel Scavenge 收集器

Parallel Scavenge 是针对新生代的垃圾回收器，采用“复制”算法，和 ParNew 类似，但更注重吞吐率。在 ParNew 的基础上演化而来的 Parallel Scanvenge 收集器被誉为“吞吐量优先”收集器。吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，即`吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）`。如虚拟机总运行了 100 分钟，其中垃圾收集花掉 1 分钟，那吞吐量就是`99%`。

Parallel Scanvenge 收集器在 ParNew 的基础上提供了一组参数，用于配置期望的收集时间或吞吐量，然后以此为目标进行收集。通过 VM 选项可以控制吞吐量的大致范围：

- `-XX：MaxGCPauseMills`：期望收集时间上限，用来控制收集对应用程序停顿的影响。
- `-XX：GCTimeRatio`：期望的 GC 时间占总时间的比例，用来控制吞吐量。
- `-XX：UseAdaptiveSizePolicy`：自动分代大小调节策略。

但要注意停顿时间与吞吐量这两个目标是相悖的，降低停顿时间的同时也会引起吞吐的降低。因此需要将目标控制在一个合理的范围中。

----

#### Parallel Old 收集器

Parallel Old 是 Parallel Scanvenge 收集器的老年代版本，多线程收集器，采用“标记-整理”算法。

| 名称                       | 说明                                               | 收集模式        | 分代适用类型  |
| -------------------------- | -------------------------------------------------- | --------------- | ------------- |
| Serial                     | 单线程串行收集器                                   | 串行收集器      | 新生代        |
| ParNew                     | 多线程并行Serial收集器                             | 并行收集器      | 新生代        |
| Parallel Scavenge          | 并行吞吐量优先收集器                               | 并行收集器      | 新生代        |
| Serial Old                 | Serial单线程收集器老年代版本                       | 串行收集器      | 老年代        |
| CMS(Concurrent Mark Sweep) | 并行最短停顿时间收集器                             | 并发收集器      | 老年代        |
| Parallel Old               | Parallel Scavenge并行收集器老年代版本              | 并行收集器      | 老年代        |
| G1                         | 面向局部收集和基于Region内存布局的新型低延时收集器 | 并发/并行收集器 | 新生代/老年代 |

```
### Serial收集器(串行收集器)
描述：单线程收集器，执行时需要暂停所有其他的工作线程，新生代采用复制算法，老年代采用标记整理算法

### Parallel New收集器(并发收集器)
描述：Serial收集器的多线程版本，除了线程数差异，其他逻辑完全一样

### Parallel Scavenge收集器(并发吞吐量收集器)
描述：相比较ParNew，更关注JVM的吞吐量，可以基于2个参数调节垃圾收集的吞吐量，-XX:MaxGCPauseMillis控制最大停顿时间，
	-XX：GCTimeRatio控制吞吐量；
		控制前者时间过小，会导致GC频繁+吞吐量下降；
		吞吐量=运行用户代码时间/(运行用户代码时间+GC时间)

### CMS收集器(最短回收停顿时间)
描述：CMS适合于服务端系统，更加关注服务的响应速度，希望系统的停顿时间越短越好；CMS基于标记清除算法做优化，分为四阶段
	- 初始标记(停止其他线程,标价关联GC ROOT的对象)
	- 并发标记
	- 重新标记(停止其他线程)
	- 并发清除
	这款收集器通过多次标记获取待清除的对象，且耗时部分都可以与用户线程并发执行具有并发收集、低停顿的特点
问题：并发期间占用CPU线程，会导致用户线程吞吐量下降；由于并发的关系，可能导致无法处理浮动垃圾；会产生内存碎片

### G1收集器(Garbage-First)
描述：并发+分代收集+内存整合+低停顿......需要再进一步研究一下JDK1.8以后版本的G1收集器工作原理
```

| 名称              | 修改参数                                           | 特点                                                         |
| ----------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| Serial            | -XX:+UseSerialGC                                   | 用于新生代的单线程收集器，采用复制算法进行垃圾收集。Serial 进行垃圾收集时，所有的用户线程必须暂停（Stop The World）。 |
| ParNew            | -XX:+UseParNewGC                                   | Serial的多线程版本，在单核CPU环境并不会比Serial更优，它默认开启的收集线程数和CPU核数，可以通过-XX:ParallelGCThreads来设置垃圾收集的线程数。 |
| Parallel Scavenge | -XX:+UseParallelGC jdk1.7、jdk1.8 新生代默认使用   | 用于新生代的多线程收集器，ParNew的目标是尽可能缩短垃圾收集时用户线程的停顿时间，Parallel Scavenge的目标是达到一个可控制的吞吐量。通过-XX:MaxGCPauseMillis来设置收集器尽可能在多长时间内完成内存回收，通过-XX:GCTimeRatio来精确控制吞吐量。 |
| Serial Old        | -XX:+UseSerialOldGC                                | Serial的老年代版本，采用标记-整理算法单线程收集器。          |
| CMS               | -XX:+UseConMarkSweepGC                             | 一种以最短回收停顿时间为目标的收集器，尽量做到最短用户线程停顿时间。CMS是基于标记-清除算法，所以垃圾回收后会产生空间碎片，通过-XX:UseCMSCompactAtFullCollection开启碎片整理（默认开启）。用-XX:CMSFullGCsBeforeCompaction设置执行多少次不压缩（不进行碎片整理）的Full GC之后，跟着来一次带压缩（碎片整理）的Full GC。-XX:ParallelCMSThreads：设定CMS的线程数量。 |
| Parallel Old      | -XX:+UseParallelOldGC jdk1.7、jdk1.8老年代默认使用 | Parallel Scavenge的老年代版本，使用-XX:ParallelGCThreads限制线程数量。 |
| G1                | -XX:+UseG1GCjdk1.7以后才提供，jdk1.9默认           | 一款全新的收集器，兼顾并行和并发功能，能充分利用多CPU资源，运行期间不会产生内存碎片。通过-XX:ParallelGCThreads设置限制线程数量；-XX:MaxGCPauseMillis设置最大停顿时间。 |



### JVM常用工具

Jps

```
JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。

```

jstat

```
jstat(JVM statistics Monitoring)是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
```

jmap

```
jmap(JVM Memory Map)命令用于生成heap dump文件，如果不使用这个命令，还阔以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件。
```

jhat

```
jhat(JVM Heap Analysis Tool)命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看。在此要注意，一般不会直接在服务器上进行分析，因为jhat是一个耗时并且耗费硬件资源的过程，一般把服务器生成的dump文件复制到本地或其他机器上进行分析。
```

jstack

```
jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。
```

jconsole

```
可视化工具，监控JVM
```

jinfo

```
jinfo 用于实时查看和调整虚拟机运行参数。如传递给 Java 虚拟机的-X（即输出中的 jvm_args）、-XX参数（即输出中的 VM Flags），以及可在 Java 层面通过System.getProperty获取的-D参数（即输出中的 System Properties）。
```



### 线上JVM参数配置

常用参数

| 参数                                   | 默认值   | 描述                                                  |
| -------------------------------------- | -------- | ----------------------------------------------------- |
| -Xms<size>                             |          | 初始Java堆大小                                        |
| -Xmx<size>                             |          | 最大堆大小                                            |
| -Xss<size>                             |          | 线程堆栈大小                                          |
| -XX:SurvivorRatio                      | 默认为8  | Eden区域与Survivor区域的容量比例(8=8：1：1)           |
| -XX:MaxGCPauseMillis                   |          | 设置GC最大停顿时间（毫秒），仅适用部分收集器          |
| -XX:MaxPermSize                        | 64M      | 永久代的最大值                                        |
| -XX:HeapDumpOnOutOfMemoryError         | 关闭     | 发生内存溢出时是否生成堆快照                          |
| -XX:OnOutOfMemoryError                 |          | 内存溢出时执行指定命令                                |
| -XX:PrintConcurrentLocks               | 默认关闭 | 打印JUC锁的状态                                       |
| -XX:PrintGC                            | 默认关闭 | 打印GC信息                                            |
| -XX:PrintGCDetails                     | 默认关闭 | 打印GC明细信息                                        |
| -XX:-PrintGCTimeStamps                 | 默认关闭 | 打印GC停顿耗时                                        |
| -XX:MaxTenuringThreshold               | 15       | 晋升到老年代的对象年龄，每进行过一次Minor GC，年龄加1 |
| -Server                                |          | 与-client相对，表示VM启动是以客户端还是服务端         |
| -XX:NewRatio                           | 2        | 老年代与新生代内存比例，默认为2:1，即1/3和2/3         |
| -XX:HeapDumpPath=./java_pid<pid>.hprof |          | 堆内存快照的存储路径                                  |
| **-XX:MaxMetaspaceSize**               |          | 元空间内存分配                                        |
|                                        |          |                                                       |

行为参数

| 参数                         | 默认值 | 描述                                              |
| ---------------------------- | ------ | ------------------------------------------------- |
| -XX:-AllowUserSignalHandlers | 不启用 | 允许用户在应用中捕捉信号 (只和Solaris和Linux有关) |


​	
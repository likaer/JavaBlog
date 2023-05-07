## 问题排查



## Java内存溢出OutOfMemoryError的产生与排查

### 可能存在的原因

1. 内存中加载的数据量过于庞大，如一次从数据库取出过多数据；
2. 集合类中有对对象的引用，使用完后未清空，使得JVM不能回收；
3. 代码中存在死循环或循环产生过多重复的对象实体；
4. 使用的第三方软件中的BUG；
5. 启动参数内存值设定的过小；



### 常规处理方案

1. 在JVM参数中增加`-XX:HeapDumpOnOutOfMemoryError`和`-XX:OnOutOfMemoryError`参数获取报错日志和内存溢出场景下的内存数据
2. 根据日志判断内存溢出的区域是哪一块(新生代，老年代，永久代等)，从而大致判断导致溢出的原因
3. 使用**`jvisualvm`**工具查看内存数据的日志，分析占用内存比例最大的对象，加快定位导致内存溢出的原因



## 部署JAVA应用的机器CPU过高的产生与排查

### 可能存在的原因

1. 频繁的GC; 如果访问量很高，可能会导致频繁的GC甚至FGC。当调用量很大时，内存分配将如此之快以至于GC线程将连续执行，这将导致CPU飙升。
2. 频繁的线程上下文切换; 有许多已启动的线程，这些线程的状态在Blocked(锁定等待，IO等待等)和Running之间发生变化。当锁争用激烈时，这种情况很容易发生。



### 常规处理方案

1. 根据`top`命令，发现PID为13033的Java进程占用CPU 50%以上，占用CPU过高
2. 找到该进程后，可以通过``shift+H``显示线程列表,也可以使用``ps``命令按照CPU占用高的线程排序`ps -mp pid -o THREAD,tid,time | sort –rn`
3. 将占用CPU最高的线程id转为16进制：`printf "%x\n" 28358`
4. 使用`jstack`获取线程堆栈，找到占用CPU最高的线程ID, 根据方法锁定代码块
5. 也可以适当的使用`jmap`和GC日志辅助分析







## 常用命令

```
tar -czvf dump.tar.gz *.log
```

Here’s what those switches actually mean:

- -c: **C**reate an archive.
- -z: Compress the archive with g**z**ip.
- -v: Display progress in the terminal while creating the archive, also known as “**v**erbose” mode. The v is always optional in these commands, but it’s helpful.
- -f: Allows you to specify the **f**ilename of the archive.



```
tar -xzvf archive.tar.gz
tar -xzvf archive.tar.gz -C /tmp
```



```
## 只显示堆中对象的统计信息
jmap -histo pid > class.out
## 生成堆转储快照dump文件
jmap -dump:format=b,file=heapdump.hprof pid
## 线程堆栈
jstack pid > thread.out
```




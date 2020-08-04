---
title: 高性能MySQL读后感-数据库引擎与事务
date: 2019-01-07 20:31:53
tags: MySQL
---


## InnoDB和MyISAM差异性

| Feature | InnoDB | MyISAM |
| ------- | ------ | ------ |
| B-Tree索引 | Yes | Yes |
| 备份恢复 | Yes | Yes |
| <font color='red'>聚簇索引</font> | Yes | No |
| 压缩数据 | Yes | Yes |
| <font color='red'>数据缓存</font> | Yes | No |
| 数据加密 | Yes | Yes |
| <font color='red'>外键</font> | Yes | No |
| 全文索引 | Yes | Yes |
| 索引缓存 | Yes | Yes |
| <font color='red'>锁粒度</font> | Row | Table |
| <font color='red'>MVCC(多版本并发控制)</font> | Yes | No |
| 复制 | Yes | Yes |
| <font color='red'>存储限制</font> | 64TB | 256TB |
| T-Tree索引 | No | No |
| <font color='red'>**事务**</font> | <font color='red'>Yes</font> | <font color='red'>No</font> |

--------------------------------

## B-Tree索引

### B-Tree定义

B树，是一种多路搜索树：

1.定义任意非叶子结点最多只有M个儿子；且M>2；

2.根结点的儿子数为[2, M]；

3.除根结点以外的非叶子结点的儿子数为[M/2, M]；

4.每个结点存放至少M/2-1（取上整）和至多M-1个关键字；（至少2个关键字）

5.非叶子结点的关键字个数=指向儿子的指针个数-1；

6.非叶子结点的关键字：K[1], K[2], …, K[M-1]；且K[i] < K[i+1]；

7.非叶子结点的指针：P[1], P[2], …, P[M]；其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其它P[i]指向关键字属于(K[i-1], K[i])的子树；

8.所有叶子结点位于同一层；


如下图所示为一个3阶的B-Tree： 
![B-Tree](/images/B-Tree.png)

每个节点占用一个盘块的磁盘空间，一个节点上有两个升序排序的关键字和三个指向子树根节点的指针，指针存储的是子节点所在磁盘块的地址。两个关键词划分成的三个范围域对应三个指针指向的子树的数据的范围域。以根节点为例，关键字为17和35，P1指针指向的子树的数据范围为小于17，P2指针指向的子树的数据范围为17~35，P3指针指向的子树的数据范围为大于35。

模拟查找关键字29的过程：

根据根节点找到磁盘块1，读入内存。【磁盘I/O操作第1次】
比较关键字29在区间（17,35），找到磁盘块1的指针P2。
根据P2指针找到磁盘块3，读入内存。【磁盘I/O操作第2次】
比较关键字29在区间（26,30），找到磁盘块3的指针P2。
根据P2指针找到磁盘块8，读入内存。【磁盘I/O操作第3次】
在磁盘块8中的关键字列表中找到关键字29。
分析上面过程，发现需要3次磁盘I/O操作，和3次内存查找操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而3次磁盘I/O操作是影响整个B-Tree查找效率的决定因素。

### 总结

- 在二叉树的基础上尽量降低树的高度，对每个节点内的数据进行二分查询，保证性能与二叉树完全一致的同时<font color='red'>降低磁盘IO的操作</font>

<font color='red'>注：MySQL的InnoDB实际上使用的是B+Tree数据结构，即在每个叶子节点上包含下一个叶子节点的指针</font>

--------------------------------

## 聚簇索引

聚簇索引并非一种单独的索引类型，而是一种数据存储方式，**聚簇表示其索引键值和数据行存储在一起**

因为设计中同一份数据不可能同时存储在多个地方中，所以一张表只能有一个聚簇索引。

- 在定义主键的情况下，InnoDB会选择主键作为聚簇索引
- 如果没有这样的索引，InnoDB会隐式定义一个主键作为聚簇索引


## 锁机制


### 读写锁

- MyISAM的锁采用表锁机制，并发情况下的读没有问题，但是并发插入性能会差一些
- InnoDB采用与Oracle类似的锁机制，提供一致性的非锁定读、 行级锁支持

InnoDB锁机制：
InnoDB存储引擎实现了两种标准的行级锁：
- 共享锁：又叫读锁，指当前事务对该行数据加读锁，且阻塞其他事务写该数据
- 排他锁：又叫写锁，指当前事务对该行数据加写锁，且阻塞其他事务读写该行数据

共享锁与排他锁的兼容性如下：

| | 共享锁 | 排他锁 |
| ------- | ------ | ------ |
| 共享锁 | 兼容 | 不兼容 |
| 排他锁 | 不兼容 | 不兼容 |

从表中可以看出来，除了共享锁和共享锁兼容，其他锁之间都不兼容

MySQL显性的加锁方法

共享锁
```
SELECT ... LOCK IN SHARE MODE;
```
排他锁
```
SELECT ... FOR UPDATE;
```

### 意向锁

InnoDB存储引擎支持多粒度的锁定，这种锁定允许事务在行级上的锁和表级上的锁同时存在

| | 意向共享锁 | 意向排他锁 | 共享锁 | 排他锁 |
|---|---|---|---|---|
| 意向共享锁 | 兼容 | 兼容 | 兼容 | 不兼容 |
| 意向排他锁 | 兼容 | 兼容 | 不兼容 | 不兼容 |
| 共享锁 | 兼容 | 不兼容 | 兼容 | 不兼容 |
| 排他锁 | 不兼容 | 不兼容 | 不兼容 | 不兼容 |

### 锁算法

InnoDB如果没法命中索引，会采用表锁，如果命中索引，则会存在3种行锁算法，其分别是
- Record Lock: 单个行记录的锁
- Gap Lock: 间隙锁，锁定一个范围，但不包含记录本身
- Next-Key Lock: 前两个锁的结合，锁定一个范围+记录本身


例如对如下SQL来说

| ID(primary key) | columnA(key) |
| --- | --- |
| 1 | 1 |
| 2 | 1 |
| 3 | 3 |
| 4 | 7 |
| 5 | 13 |

```
select * from tableA where columnA = 3 for update;
```
如果columnA是聚簇索引或者唯一索引，则InnoDB会采用Record Lock锁定单行。
如果columnA是普通索引，则InnoDB会采用Next-Key Lock锁定区间范围。
假定columnA是普通索引且索引的区间范围分为以下
(-无穷, 1)
[1, 3)
[3, 7)
[7, 13)
[13, +无穷]

Next-Lock会锁定3(记录本身)+2个索引区间范围【(1, 3), (3, 7)】，同时由于tableA存在一个聚簇索引，需要同时对聚簇索引加一个Record Lock

Gap Lock则是在无法确定记录的情况下去锁定一个范围值

因此，对于以上的操作, 以下的操作都会被堵塞
```
select * from tableA where id = 3 lock in share mode;
insert into tableA select 6, 2;
insert into tableA select 7, 6;
```

### 锁排查

在INFORMATION_SCHEMA架构下可以通过INNODB_TRX, INNODB_LOCKS, INNODB_LOCK_WAITS三张表观察当前事务并分析可能存在的锁问题

INNODB_TRX

| Column_Name | Description |
| --- | --- |
| trx_id | 唯一的事务ID号 |
| trx_state | 事务执行的状态, 允许的值为 RUNNING, LOCK WAIT, ROLLING BACK, COMMITTING. |
| trx_started | 事务开始时间 |
| trx_requested_lock_id | 等待事务的锁ID。如果trx_state是lock wait,那么该值代表当前事务等待之前事务占用锁资源的ID，关联INNODB_LOCKS;否则为Null; |
| trx_weight | 一个事务的权重,反映(不准确)改变的记录数和被事务锁定的记录数。发生死锁时, InnoDB 会选择权重最小的事务回滚. |
| trx_mysql_thread_id | MySQL中的线程ID，SHOW PROCESSLIST显示的结果 |
| trx_query | 事务执行的语句 |
| 其他 | https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-trx-table.html |

INNODB_LOCKS

![INNODB_LOCKS](/images/INNODB_LOCKS.png)

INNODB_LOCK_WAITS

| Column_Name | Description |
| --- | --- |
| requesting_trx_id | 申请锁资源的事务ID |
| requested_lock_id | 申请的锁ID |
| blocking_trx_id | 阻塞的事务ID |
| blocking_lock_id | 阻塞的锁ID |

查看哪个事务阻塞了另一个事务
```
select r.trx_id waiting_trx_id,
r.trx_mysql_thread_id waiting_thread,
r.trx_query waiting_query,
b.trx_id blocking_trx_id,
b.trx_mysql_thread_id blocking_thread,
b.trx_query blocking_query
from `information_schema`.innodb_lock_waits w inner join `information_schema`.innodb_trx b
on b.trx_id = w.blocking_trx_id
inner join `information_schema`.innodb_trx r on r.trx_id = w.requesting_trx_id;
```

--------------------------------

## InnoDB

简述：在MySQL 5.7中，InnoDB是默认的MySQL存储引擎，也是目前MySQL使用的最广泛的存储引擎

- 它的DML(Data Manipulation Language)操作遵循ACID模型，具有提交，回滚和崩溃恢复功能的事务来保护用户数据。

- 行级锁定和Oracle风格的一致性读取可提高多用户并发性和性能。

- InnoDB表格将您的数据排列在磁盘上，以根据主键优化查询。每个InnoDB表都有一个称为聚簇索引的主键索引，用于组织数据以最小化主键查找的I/O，其单表存储上限为64TB

- 要保持数据完整性，请InnoDB支持FOREIGN KEY约束。使用外键，将检查插入，更新和删除，以确保它们不会导致不同表之间的不一致。

- 支持数据备份/恢复，数据压缩，数据缓存，数据加密等

### MySQL的事务

简述：MySQL事务遵循ACID模型

## Atomicity(原子性)
指事务中的所有操作要么全部执行，要么全部不执行

- MySQL默认COMMIT和ROLLBACK的方式保证事务的一致性
- MySQL通过SQL语法支持事务的提交和回滚
```
START TRANSACTION
    [transaction_characteristic [, transaction_characteristic] ...]

transaction_characteristic: {
    WITH CONSISTENT SNAPSHOT
  | READ WRITE
  | READ ONLY
}

BEGIN [WORK]
COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
SET autocommit = {0 | 1}
```

## Consistency(一致性)
事务中的数据总是从一个一致的状态到另一个一致的状态

- [InnoDB doublewrite buffer(双写缓冲区)](https://www.cnblogs.com/geaozhang/p/7241744.html)
- InnoDB crash recovery(数据恢复)

## Isolation(隔离性)
隔离性是对事务并发过程中数据之间互相影响的程度的定义

- MySQL事务隔离级别机制

## Durability(持久性)
指事务提交后，数据会持久化在数据库，且不会由于系统崩溃导致数据丢失。

- InnoDB doublewrite buffer, turned on and off by the innodb_doublewrite configuration option.

- Configuration option innodb_flush_log_at_trx_commit.

- Configuration option sync_binlog.

- Configuration option innodb_file_per_table.

- Write buffer in a storage device, such as a disk drive, SSD, or RAID array.

- Battery-backed cache in a storage device.

- The operating system used to run MySQL, in particular its support for the fsync() system call.

- Uninterruptible power supply (UPS) protecting the electrical power to all computer servers and storage devices that run MySQL servers and store MySQL data.

- Your backup strategy, such as frequency and types of backups, and backup retention periods.

- For distributed or hosted data applications, the particular characteristics of the data centers where the hardware for the MySQL servers is located, and network connections between the data centers.

### MySQL事务隔离级别

## 1. 脏读（READ UNCOMMITED, 未提交读）

指事务中的修改，即使没有提交，对其他事务也是可见的。

点评：由于事务间不是隔离的，因此在事务并行时问题极多，甚至会读到错误的脏数据

## 2. 不可重复读（READ COMMITED, 提交读）

指事务执行时，只能读取已经提交的事务所做的修改

点评：在事务A多次读取同一数据时，若事务B执行了`更新操作`，可能会得到不同的结果。

## 3. 可重复读（REPEATABLE READ， 幻读）：MySQL的默认事务隔离级别

指事务执行时，多次读取同一数据，得到的结果总是一样，但存在幻读。

点评：幻读是指在事务A执行多次查询时，事务B执行了`插入操作`， 此时事务A多次查询后会发现突然多了一条数据。

注：在RR级别下,幻读可以通过***select ... for update***避免

## 4. 可串行化（SERIALIZEABLE）

最高隔离级别，强制事务串行执行

点评：由于是最高锁粒度，会导致大量的超时和锁竞争问题，不适合并发性场景


------

RR 通过对 select 操作手动加 行X锁（SELECT ... FOR UPDATE 这也正是 SERIALIZABLE 隔离级别下会隐式为你做的事情）可以避免幻读，同时还需要知道，即便当前记录不存在，比如 id = 1 是不存在的，当前事务也会获得一把记录锁（因为InnoDB的行锁锁定的是索引，故记录实体存在与否没关系，存在就加 行X锁，不存在就加 next-key lock间隙X锁），其他事务则无法插入此索引的记录，故杜绝了幻读。

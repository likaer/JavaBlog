---
title: 深入Mysql之事务
date: 2020-08-20 18:27:20
tags:
---



## InnoDB

简述：在MySQL 5.7中，InnoDB是默认的MySQL存储引擎，也是目前MySQL使用的最广泛的存储引擎

- 它的DML(Data Manipulation Language)操作遵循ACID模型，具有提交，回滚和崩溃恢复功能的事务来保护用户数据。

- 行级锁定和Oracle风格的一致性读取可提高多用户并发性和性能。

- InnoDB表格将您的数据排列在磁盘上，以根据主键优化查询。每个InnoDB表都有一个称为聚簇索引的主键索引，用于组织数据以最小化主键查找的I/O，其单表存储上限为64TB

- 要保持数据完整性，请InnoDB支持FOREIGN KEY约束。使用外键，将检查插入，更新和删除，以确保它们不会导致不同表之间的不一致。

- 支持数据备份/恢复，数据压缩，数据缓存，数据加密等

- [InnoDB doublewrite buffer(双写缓冲区)](https://www.cnblogs.com/geaozhang/p/7241744.html)
- InnoDB crash recovery(数据恢复)

## MySQL的事务

简述：MySQL事务遵循ACID模型

### Atomicity(原子性)
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

### Consistency(一致性)
事务中的数据总是从一个正确的状态到另一个正确的状态

- 需要注意的是，这里强调的是状态的正确性，即事务执行后需依然满足数据库的约束性，比如unique，non-negative, 如果状态不一致了则系统可以自动撤销事务返回初始状态
- 其中应用层的一致性则需要应用通过事务原子性控制保证其数据正确性


### Isolation(隔离性)
隔离性是指多事务并发场景下，该事务提交前对其他事务不可见

### Durability(持久性)
指事务提交后，数据会持久化在数据库，且不会由于系统崩溃导致数据丢失。

## MySQL事务隔离级别

### 1. 脏读（READ UNCOMMITED, 未提交读）

指事务中的修改，即使没有提交，对其他事务也是可见的。

点评：由于事务间不是隔离的，因此在事务并行时问题极多，甚至会读到错误的脏数据

### 2. 不可重复读（READ COMMITED, 提交读）

指事务执行时，只能读取已经提交的事务所做的修改

点评：在事务A多次读取同一数据时，若事务B执行了`更新操作`，可能会得到不同的结果。

### 3. 可重复读（REPEATABLE READ， 幻读）：MySQL的默认事务隔离级别

指事务执行时，多次读取同一数据，得到的结果总是一样，但存在幻读。

点评：幻读是指在事务A执行多次查询时，事务B执行了`插入操作`， 此时事务A多次查询后会发现突然多了一条数据。

注：在RR级别下,幻读可以通过***select ... for update***避免, 原理参考Next Key Lock

### 4. 可串行化（SERIALIZEABLE）

最高隔离级别，强制事务串行执行

点评：由于是最高锁粒度，会导致大量的超时和锁竞争问题，不适合并发性场景


------

RR 通过对 select 操作手动加 行X锁（SELECT ... FOR UPDATE 这也正是 SERIALIZABLE 隔离级别下会隐式为你做的事情）可以避免幻读，同时还需要知道，即便当前记录不存在，比如 id = 1 是不存在的，当前事务也会获得一把记录锁（因为InnoDB的行锁锁定的是索引，故记录实体存在与否没关系，存在就加 行X锁，不存在就加 next-key lock间隙X锁），其他事务则无法插入此索引的记录，故杜绝了幻读。
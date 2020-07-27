---
title: 高性能Mysql读后感-数据库引擎与事务
date: 2019-01-07 20:31:53
tags: Mysql
---


### Mysql的数据库引擎

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

## InnoDB

简述：在MySQL 5.7中，InnoDB是默认的MySQL存储引擎，也是目前Mysql使用的最广泛的存储引擎

- 它的DML(Data Manipulation Language)操作遵循ACID模型，具有提交，回滚和崩溃恢复功能的事务来保护用户数据。

- 行级锁定和Oracle风格的一致性读取可提高多用户并发性和性能。

- InnoDB表格将您的数据排列在磁盘上，以根据主键优化查询。每个InnoDB表都有一个称为聚簇索引的主键索引，用于组织数据以最小化主键查找的I/O，其单表存储上限为64TB

- 要保持数据完整性，请InnoDB支持FOREIGN KEY约束。使用外键，将检查插入，更新和删除，以确保它们不会导致不同表之间的不一致。

- 支持数据备份/恢复，数据压缩，数据缓存，数据加密等

### Mysql的事务

简述：Mysql事务遵循ACID模型

## Atomicity(原子性)
指事务中的所有操作要么全部执行，要么全部不执行

- Mysql默认COMMIT和ROLLBACK的方式保证事务的一致性
- Mysql通过SQL语法支持事务的提交和回滚
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

- Mysql事务隔离级别机制

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

### Mysql事务隔离级别

## 1. 脏读（READ UNCOMMITED, 未提交读）

指事务中的修改，即使没有提交，对其他事务也是可见的。

点评：由于事务间不是隔离的，因此在事务并行时问题极多，甚至会读到错误的脏数据

## 2. 不可重复读（READ COMMITED, 提交读）

指事务执行时，只能读取已经提交的事务所做的修改

点评：在事务A多次读取同一数据时，若事务B执行了`更新操作`，可能会得到不同的结果。

## 3. 可重复读（REPEATABLE READ， 幻读）：Mysql的默认事务隔离级别

指事务执行时，多次读取同一数据，得到的结果总是一样，但存在幻读。

点评：幻读是指在事务A执行多次查询时，事务B执行了`插入操作`， 此时事务A多次查询后会发现突然多了一条数据。

## 4. 可串行化（SERIALIZEABLE）

最高隔离级别，强制事务串行执行

点评：由于是最高锁粒度，会导致大量的超时和锁竞争问题，不适合并发性场景


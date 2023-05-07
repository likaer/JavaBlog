# 高性能MySQL实战(8.0)



查询性能优化

事务角度看待MySQL-InnoDB



### 索引的数据结构

![image-20211220154848838](/Users/likaer/Library/Application Support/typora-user-images/image-20211220154848838.png)

```SQL
CREATE TABLE `te_test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `Col1` bigint(20) NOT NULL,
  `Col2` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_col2` (`Col2`) USING BTREE,
  KEY `idx_col1_col2` (`Col1`,`Col2`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='demo表';

select * from te_test order by id;
```

![屏幕快照 2021-12-20 下午3.34.10](/Users/likaer/Desktop/屏幕快照 2021-12-20 下午3.34.10.png)



![img](https://cdn.spirithy.com/static/images/hosting_by_mweb/20191110_sql02ci.jpg)

![img](https://cdn.spirithy.com/static/images/hosting_by_mweb/20191110_sql03si.jpg)

![img](https://cdn.spirithy.com/static/images/hosting_by_mweb/20191110_sql04sim.jpg)





## 查询性能优化

### 查询的生命周期



![图片1](/Users/likaer/Desktop/图片1.png)

客户端->服务端->SQL解析->生成执行计划->执行

整个生命周期中，其中**执行**是整个生命周期中最影响性能的阶段。

1. SQL拆解

要优化一条查询SQL，我们需要学会如何去拆解我们的查询SQL，将一条SQL拆解为多个子任务，那么每个子任务都会消耗一定的时间，而我们需要找到那些不必要的子任务，或减少某些子任务的执行耗时，或减少子任务的执行次数

```SQL
CREATE TABLE `uuc_business_user` (
	`id` bigint(20) UNSIGNED NOT NULL COMMENT '主键',
	`password` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '密码',
	`phone` char(15) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '手机号（账号）',
	`nickname` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '昵称、别名',
	`name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '真实姓名',
  `tag` json DEFAULT NULL COMMENT '标签：json串，方便上新功能给指定用户使用',
	`gender` tinyint(2) DEFAULT NULL COMMENT '性别 0未知 1男 2女',
	`type` tinyint(4) DEFAULT '0' COMMENT '0游客，1职员,2加盟商',
	`source` tinyint(4) NOT NULL DEFAULT '10' COMMENT '用户来源，10-系统添加，20-微信小程序，30-美团，40-饿了么，50-哗啦啦，60-支付宝，70-钉钉',
	`avatar` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '头像',
	`remark` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '备注',
	`ding_user_id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '钉钉的userid',
	`job_number` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '钉钉的工号',
	`ding_json` json DEFAULT NULL COMMENT '钉钉json详情',
	PRIMARY KEY (`id`),
	UNIQUE KEY `uniq_phone` (`phone`),
	UNIQUE KEY `uniq_dingUserId` (`ding_user_id`),
	KEY `idx_job_number` (`job_number`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 DEFAULT COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = DYNAMIC COMMENT 'B端用户表'  ;
```

```SQL
select * from uuc_business_user where name like '刘%' order by phone;
```

- 查询字段
- where查询条件
- 子SQL，JOIN SQL等
- group by, order by



2. 衡量子任务的指标

- 响应时间(参考指标)
- 扫描行数
- 返回行数



3. 索引扫描类型

- 全表(all)

  - ```explain select * from uuc_business_user where name like '刘%' order by phone;```

- 使用索引进行全表扫描(index)

  - 索引是查询的覆盖索引
  - 使用从索引中读取来执行全表扫描以按索引顺序查找数据行
  - ```explain select phone from uuc_business_user order by phone;```

- 有范围的索引扫描(range)

  - 当使用[`=`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal), [`<>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-equal), [`>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than), [`>=`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than-or-equal), [`<`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than), [`<=`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than-or-equal), [`IS NULL`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-null), [`<=>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal-to), [`BETWEEN`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_between), [`LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like), 或 [`IN()`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_in)等运算符中的任何一个将键列与常量进行比较
  - ```explain select * from uuc_business_user where phone like '139%' order by phone;```

- 使用了索引扫描，且索引列存在重复值(ref)

  - ```explain select * from uuc_usergroup where name = '温岭大溪一店';```

- 使用了主键或唯一索引扫描(eq_ref)

  - 当连接使用索引的所有部分并且索引是一个 `PRIMARY KEY`或`UNIQUE NOT NULL`索引时使用它
  - [`eq_ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_eq_ref)可用于使用`=`运算符进行比较的索引列 。比较值可以是常量或表达式
  - ```explain select * from uuc_business_user, uuc_user where uuc_business_user.id = uuc_user.id;```

- 常数引用(const)

  - 将 `PRIMARY KEY`或 `UNIQUE`索引的所有部分与常量值进行比较时使用
  - ```explain select * from uuc_business_user where id = 5351;```

  

4. MySQL如何使用索引

- `WHERE`快速 查找匹配的行。
- **如果有多个索引之间的选择，MySQL通常使用找到最少行数的索引**
  - 需要注意的是，MySQL预估的最优索引并不总是准确
- 如果表具有多列索引，则优化器可以使用索引的任何最左边的前缀来查找行。举例来说，如果你有一个三列的索引 `(col1, col2, col3)`，你有索引的搜索功能`(col1)`， `(col1, col2)`以及`(col1, col2, col3)`
- 如果```order by```或```group by```是在可用索引的最左侧前缀上完成的，则使用索引扫描
- 当查询需要访问大部分行时，顺序读取比通过索引读取要快。





5. [EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/explain-output.htmlv)

| Column                                                       | JSON Name       | Meaning                                        |
| :----------------------------------------------------------- | :-------------- | :--------------------------------------------- |
| [`id`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_id) | `select_id`     | The `SELECT` identifier                        |
| [`select_type`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_select_type) | None            | The `SELECT` type                              |
| [`table`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_table) | `table_name`    | The table for the output row                   |
| [`partitions`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_partitions) | `partitions`    | The matching partitions                        |
| [`type`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_type) | `access_type`   | The join type                                  |
| [`possible_keys`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_possible_keys) | `possible_keys` | The possible indexes to choose                 |
| [`key`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key) | `key`           | The index actually chosen                      |
| [`key_len`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key_len) | `key_length`    | The length of the chosen key                   |
| [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_ref) | `ref`           | The columns compared to the index              |
| [`rows`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_rows) | `rows`          | Estimate of rows to be examined                |
| [`filtered`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_filtered) | `filtered`      | Percentage of rows filtered by table condition |
| [`Extra`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_extra) | None            | Additional information                         |



```
explain select * from uuc_usergroup where code = 16 and client_id = 1000031;
show warnings;
```





6. 实战SQL

```SQL
####shop-prod
explain select * from shop where merchant_id not in (
	select id from merchant where id < 1472857786005164035
) and merchant_id != 0;

explain select * from shop where merchant_id in (
	select id from merchant where id >= 1471857786005164035
);

explain select * from shop where `merchant_id` >= 1471857786005164035;
```







```SQL
####user_center_ding
explain select * from uuc_business_user where phone > 19000000000;

explain select job_number from uuc_business_user where left(`job_number`, 3) = 'G000';

explain select distinct job_number from uuc_business_user where job_number like 'A00%' or `job_number` like '%003';

```











```SQL
####user_center_ding
explain select * from uuc_business_user where id >= 1290988159236050945;

explain select * from uuc_business_user where phone > '19000000000';

explain select * from uuc_business_user where id >= 1290988159236050945 and phone > '19000000000';
```





### InnoDB-事务

1. 事务隔离级别

- 脏读（READ UNCOMMITED, 未提交读）
- 不可重复读（READ COMMITED, 提交读）
- 可重复读（REPEATABLE READ， 幻读）Mysql的默认事务隔离级别
- 可串行化（SERIALIZEABLE）



| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读                                      |
| ---------------------------- | ---- | ---------- | ----------------------------------------- |
| 读未提交（read-uncommitted） | 是   | 是         | 是                                        |
| 不可重复读（read-committed） | 否   | 是         | 是                                        |
| 可重复读（repeatable-read）  | 否   | 否         | <font color="red">特定场景下会发生</font> |
| 串行化（serializable）       | 否   | 否         | 否                                        |



2. MySQL事务执行

```SQL
START TRANSACTION
    [transaction_characteristic [, transaction_characteristic] ...]
BEGIN [WORK]
COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
SET autocommit = {0 | 1}
```

READ COMMITED

```SQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
INSERT INTO `te_temp` (`shop_id`, `name`, `modify_time`)
VALUES(1876, '葡萄', '2021-12-13 18:30:31');
update te_temp set name = '葡萄1' where shop_id = 1876 and name = '葡萄'
COMMIT;
```

```SQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT * FROM te_temp WHERE SHOP_iD = 1876;
COMMIT;
```

REPEATABLE READ

```SQL
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
INSERT INTO `te_temp` (`shop_id`, `name`, `modify_time`)
VALUES(1876, '葡萄', '2021-12-13 18:30:31');
COMMIT;
```

```SQL
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT * FROM te_temp where shop_id = 1876;
update te_temp set name = '四季青1' where shop_id = 1876 and name = '四季青';
SELECT * FROM te_temp where shop_id = 1876;
update te_temp set name = '葡萄1' where shop_id = 1876 and name = '葡萄';
SELECT * FROM te_temp where shop_id = 1876;
COMMIT;
```



3. 多版本并发控制(MVCC)

| 时刻 | 事务10                                            |
| :--- | :------------------------------------------------ |
| 1    | SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;  |
| 2    | BEGIN;                                            |
| 3    | SELECT * FROM students WHERE id = 99;             |
| 4    | UPDATE students SET name = 'Alice' WHERE id = 99; |
| 5    | SELECT * FROM students WHERE id = 99;             |
| 6    | COMMIT;                                           |



| **数据列(name)** | **DB_ROW_ID** | **DB_TRX_ID** | **DB_ROLL_PTR** |
| ---------------- | ------------- | ------------- | --------------- |
| Alice            | 99            | 10            | 0x104           |



| ReadView-字段    | 当前值     | ReadView-描述              |
| ---------------- | ---------- | -------------------------- |
| m_low_limit_id   | 13         | 大于等于该ID的事务均不可见 |
| m_up_limit_id    | 8          | 小于这个ID的事务均可见     |
| m_creator_trx_id | 10         | 当前事务ID                 |
| m_ids            | [8, 11,12] | 创建时活跃的事务ID列表     |

7----？

8----？

9----？

10----？

11----？

12----？

13----？



UNDO_LOG

| **name** | **DB_ROW_ID** | **DB_TRX_ID** | **DB_ROLL_PTR** |
| -------- | ------------- | ------------- | --------------- |
| Alice    | 99            | 10            | 0x103           |

​																																										⬇️

| **name** | **DB_ROW_ID** | **DB_TRX_ID** | **DB_ROLL_PTR** |
| -------- | ------------- | ------------- | --------------- |
| Bob      | 99            | 5             | 0x99            |

​																																										⬇️

| **name** | **DB_ROW_ID** | **DB_TRX_ID** | **DB_ROLL_PTR** |
| -------- | ------------- | ------------- | --------------- |
| Bob      | 99            | 2             | null            |






# 高性能MySQL实战(8.0)



**MySQL数据类型**
**如何创建高性能索引**
SQL查询优化
数据库引擎与事务





## MySQL历史

| **version**                                  | **about**                                  |
| -------------------------------------------- | ------------------------------------------ |
| 3.11.1（1996）                               | First release                              |
| 4.0   （2003）                               | 查询缓存, UNION, 全文索引, InnoDB          |
| 4.1   （2004-10）                            | 子查询, R-trees索引                        |
| 5.0   （2005-10）                            | 存储过程, 视图, 游标, 触发器, 分布式事务   |
| 5.1   （2008-11）                            | 分区, Plugin API, InnoDB Plugin, MySQL集群 |
| 5.5  GA版本5.5.8 （2010-12-03）              | 默认InnoDB plugin引擎                      |
| 5.6  GA版本5.6.10（2013-02-05）              | InnoDB支持全文索引                         |
| 5.7  （2013-02） GA版本5.7.10 （2015-12-07） | JSON支持                                   |





##### 

```sql
CREATE TABLE `test` (
  `id` tinyint(4) NOT NULL COMMENT 'ID'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```



<font color="Red">**CHARSET和COLLATE的含义？**</font>



-----



















## MySQL字符集、排序规则

### CHARSET的含义

charset是一类字符的编码集合，不同的字符集存储的字符数量，采用的编码规则和存储方案都不同，

常见字符集名称：ASCII字符集、GB2312字符集、 GB18030字符集、Unicode字符集等

```sql
show CHARACTER set;
```

| charset | description            | default_collation  | max_len |
| ------- | ---------------------- | ------------------ | ------- |
| utf8    | UTF-8 Unicode          | utf8_general_ci    | 3       |
| utf8mb4 | UTF-8 Unicode          | utf8mb4_0900_ai_ci | 4       |
| gbk     | GBK Simplified Chinese | gbk_chinese_ci     | 2       |
| ascii   | US ASCII               | ascii_general_ci   | 1       |

### COLLATE的含义

------

COLLATE则是MySQL的一种排序规则，排序规则会影响到排序，比较大小等因素；

最简单的比较规则就是比较字符集内编码的先后顺序，那么根据ASCII规则，我们可以认为1<2, a<b, 

那如果涉及到a和A之间的比较时候，我们可能希望不区分大小写，这时候就需要别的规则来定义。

```
show collation like 'utf8%';
```

| collation             | charset | id   | default | pad Attribute |
| --------------------- | ------- | ---- | ------- | ------------- |
| utf8mb4_0900_ai_ci    | utf8mb4 | 255  | Yes     | NO PAD        |
| utf8mb4_0900_as_ci    | utf8mb4 | 305  |         | NO PAD        |
| utf8mb4_0900_as_cs    | utf8mb4 | 278  |         | NO PAD        |
| utf8mb4_0900_bin      | utf8mb4 | 309  |         | NO PAD        |
| utf8mb4_bin           | utf8mb4 | 46   |         | PAD SPACE     |
| utf8mb4_cs_0900_ai_ci | utf8mb4 | 266  |         | NO PAD        |
| utf8mb4_cs_0900_as_cs | utf8mb4 | 289  |         | NO PAD        |

- Pad_attribute: MySQL 比较字符串时尾部空格是否忽略
  - NO PAD： 排序规则在比较中将尾随空格视为重要的，就像任何其他字符一样。
  - PAD SPACE：尾随空格在比较中无关紧要；字符串在不考虑尾随空格的情况下进行比较。

| 后缀   | 意义                                   |
| :----- | :------------------------------------- |
| `_ai`  | 口音不敏感(e，è，é，ê和ë之间没有区别)  |
| `_as`  | 口音敏感                               |
| `_ci`  | 不区分大小写                           |
| `_cs`  | 区分大小写                             |
| `_ks`  | 假名敏感(日语排序规则，片假名和平假名) |
| `_bin` | 二进制                                 |



***字符集和排序规则有四个级别的默认设置：服务器、数据库、表和列***



<font color="red">***MySQL我们最常用的日期数据类型是DATETIME, 那么在MySQL8.0中该类型支持的最小精度?***</font>









## MySQL数据类型



### 数字

-------

整数类型（精确值）——INTEGER、INT、SMALLINT、TINYINT、MEDIUMINT、BIGINT
定点类型（精确值）- DECIMAL
浮点类型（近似值）- FLOAT、DOUBLE
位值类型 - BIT

----

#### 整数类型

| Type      | Storage (Bytes) | Minimum Value Signed | Minimum Value Unsigned | Maximum Value Signed | Maximum Value Unsigned |
| --------- | --------------- | -------------------- | ---------------------- | -------------------- | ---------------------- |
| TINYINT   | 1               | -128                 | 0                      | 127                  | 255                    |
| SMALLINT  | 2               | -32768               | 0                      | 32767                | 65535                  |
| MEDIUMINT | 3               | -8388608             | 0                      | 8388607              | 16777215               |
| INT       | 4               | -2147483648          | 0                      | 2147483647           | 4294967295             |
| BIGINT    | 8               | -2^63                | 0                      | 2^63-1               | 2^64-1                 |

```sql
CREATE TABLE `test` (
  `num1` tinyint(4) NOT NULL COMMENT 'num1',
  `num2` tinyint(2) unsigned NOT NULL COMMENT 'num2'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```





```SQL
//创建表
CREATE TABLE `test` (
  `num1` tinyint(4) NOT NULL COMMENT 'num1'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

select @@sql_mode;###STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

insert into test(num1)values(555);###Out of range value for column 'num1' at row 1

set sql_mode = '';
insert into test(num1)values(555);
```

  **当超出范围的值分配给整数列时，MySQL 存储表示列数据类型范围的相应极值的值。**

----

#### Decimal

```sql
CREATE TABLE `test` (
  salary DECIMAL(5,2) COMMENT '工资'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```



- 5是精度，2 是刻度。
- 最大值是999.99， 最小值是-999.99
- 精度的取值范围为1〜65，刻度的取值范围为0~30
- 刻度应该是<=精度



<font color="red">在对小数进行精确计算和存储时，应尽量使用DECIMAL---如财务数据，交易数据.</font>

**当为浮点或定点列分配的值超出指定精度和刻度位数所隐含的范围时，MySQL 存储表示该范围对应的极值。(小数超了，整体长度超了）**

### 日期

----

- DATE(年月日)
- TIME(时分秒)
- DATETIME
- TIMESTAMP
- YEAR

| **Type**      | **microseconds** | **Minimum Value Signed**            | **Maximum Value Signed**            | **Zero Value**          |
| ------------- | ---------------- | ----------------------------------- | ----------------------------------- | ----------------------- |
| **DATE**      | **\**            | **1000-01-01**                      | **9999-12-31**                      | **0000-00-00**          |
| **TIME**      | **Supported**    | **-838:59:59.000000**               | **838:59:59.000000**                | **00:00:00**            |
| **DATETIME**  | **Supported**    | **1000-01-01 00:00:00.000000**      | **9999-12-31 23:59:59.999999**      | **0000-00-00 00:00:00** |
| **TIMESTAMP** | **Supported**    | **(UTC)1970-01-01 00:00:01.000000** | **(UTC)2038-01-19 03:14:07.999999** | **0000-00-00 00:00:00** |
| **YEAR**      | **\**            | **1901**                            | **2155**                            | **0000**                |



```sql
CREATE TABLE `test` (
  create_time DATETIME(3) DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
  update_time DATETIME(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

零值是在非严格SQL的模式下才会存在的，有时候会用零值代替NULL

日期的越界分为几种不同的情况

- 某个部位越界
- 整体越界





### 字符串

-----

- VARCHAR和CHAR是两种最主要的字符串类型
-  BLOB和TEXT则是为存储大数据而设计的字符串
- BINARY 和 VARBINARY 是类似VARCHAR和CHAR的两种二进制字符串类型，差异点在于他们存储字符串的二进制形态

- ENUM和SET类型





#### VARCHAR和CHAR

- varchar
   - VARCHAR(0~65535)
   - 存储可变长字符串，需要使用1-2个字节额外记录字符串长度(字符串超过255字节)
 - char
    - CHAR(0~255)
    - 存储定长字符串，因此不容易产生碎片
    - <font color="red">存储CHAR类型时，MySQL会针对空格做特殊处理</font>





```SQL
CREATE TABLE `test` (
  `str` char(4) NOT NULL COMMENT 'str'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

insert into test(str)values('1234');
insert into test(str)values('1234 ');
insert into test(str)values('123');
insert into test(str)values('123 ');
insert into test(str)values(' 123');
insert into test(str)values(' 1234');

```

**由于VARCHAR使用的是不定长字符串，因为UPDATE可能使字符串变长时，这就需要做额外的工作，因为在高频UPDATE场景下，使用CHAR会比VARCHAR更合适**





- BLOB：
  - TINYBLOB，BLOB，MEDIUMBLOB，LONGBLOB
  -  BLOB是一个二进制大对象，可以容纳可变数量的数据
- TEXT：
  - TINYTEXT，TEXT，MEDIUMTEXT，LONGTEXT
  - TEXT是纯字符串
  - TEXT索引会在末尾补充空格(唯一索引会有问题)



**不要将大字段文本存到业务实体表，而是创建另一张单独的实体表，使用ID+TEXT来关联查询，并且只在真正需要的时候查询**

​		原因是当查询的时候返回这两个字段的时候，会需要用到隐式临时表(通常使用Memory来存储)，但是memory不支持这两种类型，因此MySQL会使用MyISAM磁盘临时表，这大大拖累了性能



### 空间类型

----

- GEOMETRY
- POINT
- LINESTRING
- POLYGON





### JSON数据类型

----

```SQL
CREATE TABLE `test` (
  `json1` JSON NOT NULL COMMENT 'json'
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;


INSERT INTO test VALUES 
('{"x": 17, "x": "red"}'),
('{"x": 17, "x": "red", "x": [3, 5, 7]}');

select * from test;
{"x": "red"}
{"x": [3, 5, 7]}


select json1->"$.x" from test;
"red"
[3, 5, 7]
```

- 支持存储JSON对象和JSON数组
- 重复Key的情况下，更早的键会被丢弃







### 数据类型使用规范

```
1. 更小的通常更好
- 降低了存储空间，减少CPU和内存的开销
- 保证没有低估数据范围，扩容是一个耗时和痛苦的操作
2. 简单的更好
- 整型比字符串代价更低
- 例：用整型存储IP地址，用Date而不是字符串来存储日期和时间
3. 尽量避免NULL
- 可为NULL的列会占用更多的存储空间，在MySQL中需要很多特殊处理
- 可为NULL的列会使索引更加复杂
- 将可为NULL改为NOT NULL对性能的提升比较小，所以除非确定这会导致问题，
  否则没必要去专门修改这种情况
```







## MySQL索引



在MySQL中，索引是由存储引擎实现的， 所以相同的建表方式，底层对应的索引实现方案可能完全不同的。



#### 索引数据结构

- B-Tree(PRIMARY KEY、 UNIQUE、INDEX)
- 哈希索引(Memory引擎)
- 全文索引(FULLTEXT)

大多数 MySQL 索引（`PRIMARY KEY`、 `UNIQUE`、`INDEX`和 `FULLTEXT`）都存储在 B树。例外：空间数据类型的索引使用 R 树；`MEMORY` 表也支持[哈希索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_hash_index)；`InnoDB`对`FULLTEXT`索引使用倒排列表。



#### 哪些行为会影响你的索引

- where条件中间
- 排序条件
- 查询的列名



## B-Tree索引

可以进入索引的查询类型

- 全值匹配=
- 匹配最左前缀 like 'xxxx%'
- 匹配列前缀
- 匹配范围值
- 精确匹配某一列以及范围匹配另一列(a, b, c) select * from tableA where a = 1 and b > 2 and c > 3
- 覆盖索引(a ) select a from tableA where a > 1;



### 高性能索引策略

#### 使用独立的列来进行索引
```sql
CREATE TABLE `test` (
  `num1` bigint(20) NOT NULL COMMENT 'num1',
   `create_time` DATETIME(3) DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
   KEY `idex_num1` (`num1`) USING BTREE,
   KEY `idx_create_time`(`create_time`) USING BTREE
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

select * from test where num1 + 1 = 1;

select * from test where FROM_UNIXTIME(create_time) = '2016-04-20 20:00:00';


###前缀索引
CREATE TABLE `test` (
  `num1` bigint(20) NOT NULL COMMENT 'num1',
   `create_time` DATETIME(3) DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
   city varchar(50) NOT NULL COMMENT '城市',
   KEY `idex_num1` (`num1`) USING BTREE,
   KEY `idx_create_time`(`create_time`) USING BTREE,
  KEY `idx_city`(city(7)) USING BTREE
) ENGINE=InnoDB CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;


SELECT COUNT(DISTINCT CITY)/COUNT(*) FROM test;
SELECT COUNT(DISTINCT LEFT(CITY, 7))/COUNT(*) FROM TEST;
```



### 使用多列索引

假设有一张A表，要用(column1, column2, column3)创建一个多列索引，跟为column1, column2, column3分别创建一个索引有什么区别？

应该用什么样的顺序去排列？

- 将选择性最高的列放到索引的最前列
- 寻找searchable arguments来使用区分度更加明显的列
- 保证查询性能的情况下，尽量少的建立索引







### 聚簇索引

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20210305192305981.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dzYXN1a2U%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1637941385&t=61aa6faea6877597428656063b58c17b)





### 使用索引扫描来做排序





### 小技巧和工具

```SQL
explain select * from test where num1 + 1 = 1;

select * from `information_schema`.index_statistics;
```

| **Column**    | **JSON Name** | **Meaning**                                    |
| ------------- | ------------- | ---------------------------------------------- |
| id            | select_id     | The SELECT identifier                          |
| select_type   | None          | The SELECT type                                |
| table         | table_name    | The table for the output row                   |
| partitions    | partitions    | The matching partitions                        |
| type          | access_type   | the join type                                  |
| possible_keys | possible_keys | The possible indexes to choose                 |
| key           | key           | The index actually chosen                      |
| key_len       | key_length    | The length of the chosen key                   |
| ref           | ref           | The columns compared to the index              |
| rows          | rows          | Estimate of rows to be examined                |
| filtered      | filtered      | Percentage of rows filtered by table condition |
| Extra         | None          | Additional information                         |


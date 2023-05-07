---
title: drds
date: 2022-04-15 17:51:00
tags:
---

```shell
 mysql -h drdshbgar08q7jri.drds.aliyuncs.com -uWisdom_123 -pEsdf234kdsf -e "use wisdom_task;SHOW FULL DB STATUS like '%te_task_instance%'" > ~/out.txt
```

# PolarDB-X


## 选择合适的实例规格

| 系列  | 实例规格码 | CPU和内存最大存储容量 |  最大存储容量 |  最大连接数 |  最大IOPS |  特点 |
| ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
| 通用 |	polarx.x4.medium.2e |	2核8 GB |	3072 GB |	20000 | 	4000	|定位入门级，用于测试、体验和极小负载的场景。 | 
| 通用 |	polarx.x4.large.2e | 4核16 GB | 	3072 GB |	20000 | 	7000	| CPU和MEM配比为1:4，复用计算资源享受规模红利，性价比高。 |
| 通用 |	polarx.x4.xlarge.2e |	8核32 GB | 	3072 GB | 	20000 |	12000	| |
| 通用 |	polarx.x4.2xlarge.2e |	16核64 GB |	3072 GB |	20000 |	14000	| |
| 独享 |	polarx.x8.large.2e |	4核32  GB |	3072 GB |	20000 |	9000	|CPU和MEM配比为1:8，独占分配到的计算资源（如CPU），性能表现更加稳定。 |
| 独享 |	polarx.x8.xlarge.2e |	8核64 GB |	3072 GB |	20000 |	18000	| |
| 独享 |	polarx.x8.2xlarge.2e |	16核128 GB |	3072 GB |	20000 |	36000	| |
| 独享 |	polarx.x8.4xlarge.2e |	32核128 GB |	3072 GB |	20000 |	36000	| |
| 独享 |	polarx.x8.4xlarge.2e |	32核256 GB |	3072 GB |	20000 |	72000	| |
| 独占 |	polarx.st.8xlarge.25 |	60核470 GB |	6144 GB |	20000 |	120000	| 独占物理机规格，可以有更好的资源使用保障。 |
| 独占 |	polarx.st.12xlarge.25 |	90核720 GB |	6144 GB |	20000 |	140000	| |


 - 按IOPS选择实例规格
 	+ (类型1节点IOPS*节点数量 + 类型2节点IOPS*节点数量 + ...)*0.7 >= 峰值流量
 - 按存储容量选择
 	+ (类型1节点IOPS*最大存储容量 + 类型2节点IOPS*最大存储容量 + ...)*0.7 >= 3年内的存储容量
 - 如果出现了负载不均的情况，那么该计算方案不会完全准确

## 扩容

PolarDB-X中，扩容包含两种方式
	- 升配，指节点数不变，升级已有节点的CPU、内存、IOPS等规格（轻量级）
	- 增加节点，指节点的规格不变，增加新的计算节点与存储节点（重量级，涉及到数据re-balance）

日常运维，我们可以通过以下指标来评估是否需要扩容
CN的负载一般重点关注以下指标，
	- CPU使用率
	- 活跃线程数 (runing thread）
	- 响应时间（逻辑RT、物理RT）
DN的负载一般重点关注以下指标：
	- CPU使用率
	- IOPS使用率
	- 活跃链接数 (active session）


## 拆分函数对分库、分表的支持情况

| 拆分函数                                                     | 说明           | 是否支持用于分库 | 是否支持用于分表 |
| :----------------------------------------------------------- | :------------- | :--------------- | :--------------- |
| [HASH](https://help.aliyun.com/document_detail/71276.htm#multiTask728) | 简单取模       | 是               | 是               |
| [STR_HASH](https://help.aliyun.com/document_detail/95513.htm#multiTask4295) | 截取字符串子串 | 是               | 是               |
| [UNI_HASH](https://help.aliyun.com/document_detail/71279.htm#multiTask1115) | 简单取模       | 是               | 是               |
| [RIGHT_SHIFT](https://help.aliyun.com/document_detail/71290.htm#multiTask799) | 数值向右移     | 是               | 是               |
| [RANGE_HASH](https://help.aliyun.com/document_detail/71284.htm#multiTask833) | 双拆分列哈希   | 是               | 是               |
| [MM](https://help.aliyun.com/document_detail/71294.htm#multiTask578) | 按月份哈希     | 否               | 是               |
| [DD](https://help.aliyun.com/document_detail/71310.htm#multiTask602) | 按日期哈希     | 否               | 是               |
| [WEEK](https://help.aliyun.com/document_detail/71311.htm#multiTask623) | 按周哈希       | 否               | 是               |
| [MMDD](https://help.aliyun.com/document_detail/71332.htm#multiTask655) | 按月日哈希     | 否               | 是               |
| [YYYYMM](https://help.aliyun.com/document_detail/71334.htm#topic1014) | 按年月哈希     | 是               | 是               |
| [YYYYWEEK](https://help.aliyun.com/document_detail/71335.htm#multiTask1103) | 按年周哈希     | 是               | 是               |
| [YYYYDD](https://help.aliyun.com/document_detail/71337.htm#multiTask1081) | 按年日哈希     | 是               | 是               |





```
dbpartition by RANGE_HASH(id, source_id, 3)
```

这种情况下like 'xxx%'会导致索引失效,需警惕







```
SHOW FULL DB STATUS like '%te_task_instance%'
show full ddl;
```













## 遇见过的异常CASE




```sql
select * from te_task_instance where source_id = 1060040516106684866 limit 1,10;
同一条SQL多次执行返回的结果不一样
```

原因：在没有order by 的情况下，分库分表会基于获取数据的先后顺序进行排序，这会导致数据同一页的数据多次请求返回的结果不一样

```sql
线程A：
select * from te_task_instance where id = 1060040516106684866 for update;
update te_user_task_rel set node = '{xxxx}' where id = 1060040585220037961;
线程B：
select * from te_task_instance where id = 1060040516106684866 for update;
select * from te_user_task_rel where id = 1060040585220037961 and user_id = xxx;
现象：线程B锁等待后查询te_user_task_rel没有取到最新的node字段的信息
```

原因：DRDS采用XA事务支持分布式事务，在分布式事务场景下如果te_task_instance处在分片A，而te_user_task_rel处在分片B，在commit阶段分片A的锁释放有可能早于分片B的update，此时线程B取得分片A的锁成功后，可能查到分片B的数据是更新前的数据

相关资料

https://zhuanlan.zhihu.com/p/415744064

https://zhuanlan.zhihu.com/p/355413022







-------

```sql
select * from te_task_instance where source_id = 1537707616858529794 ---- 5004条
select * from te_task_instance where source_id = 1537707616858529794 and deaL_status = 'FINISH' ----1400+条
select * from te_task_instance where source_id = 1537707616858529794 and deaL_status = 'FINISH'  order by id desc, target_id;----报错

[ERR_EXECUTE_ON_MYSQL] Error occurs when execute on GROUP 'TASK_ENGINE_1654656028959NQAA_DVOK_0002' ATOM 'rm-bp1b884b7103m5725_task_engine_fysz_0002': Out of sort memory, consider increasing server sort buffer size

```

MySQL8.0的BUG

https://bugs.mysql.com/bug.php?id=103465







--------

```SQL
select * from te_task_instance where target_id = 1 查询到的数据量和实际数据存储的不一致;
```

原因：

1. te_task_instance采用RANG_HASH(id, target_id, 3)进行分片
2. 历史数据迁移的时候部分数据id和target_id的后3位不一致
3. 导致SQL执行的时候取后3位不能定位到正确的分区.



--------

```SQL
select id % 128, count(1) from shop group by id%128;
```

根据分区键评估数据分布情况，Twitter的分布式ID生成工具在分区方案内会出现数据倾斜
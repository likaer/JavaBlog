---
title: RocketMQ introduce
date: 2018-10-08 19:43:06
tags: RocketMQ
---


1. RocketMQ是什么？

Apache RocketMQ is an open source distributed messaging and streaming data platform.

2. 为什么需要消息框架



2. 消息中间件

(1)是一个队列模型的消息中间件，具有高性能、高可靠、高实时、分布式特点。
(2)Producer、Consumer、队列都可以分布式。
(3)Producer向一些队列轮流发送消息，队列集合称为Topic，Consumer如果做广播消费，则一个consumer实例消费这个Topic对应的所有队列，如果做集群消费，则多个Consumer实例平均消费这个topic对应的队列集合。
(4)能够保证严格的消息顺序
(5)提供丰富的消息拉取模式
(6)高效的订阅者水平扩展能力
(7)实时的消息订阅机制
(8)亿级消息堆积能力
(9)较少的依赖



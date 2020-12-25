---
title: Rocket-3
date: 2020-11-16 10:21:44
tags: RocketMQ
---


# RocketMQ

RocketMQ更专注于解决业务系统的消息交互，因此其功能针对业务系统有良好的扩展

1. 支持消息的顺序性
2. 支持Broker端的基于Tag对消息过滤
3. 消息防丢失（取决于消息是同步刷盘还是异步刷盘）
4. 消息回溯
5. 事务消息
6. 消息重试



RocketMQ架构

![Maven repo](images/rocketmq_architecture_1.png)

# 基本概念
----
## 1 消息模型（Message Model）

RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。

## 2 消息生产者（Producer）
 负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。同步和异步方式均需要Broker返回确认信息，单向发送不需要。
 
## 3 消息消费者（Consumer）
 负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。
 
## 4 主题（Topic）
  表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。

## 5 代理服务器（Broker Server）
消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。

## 6 名字服务（Name Server）
 名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。



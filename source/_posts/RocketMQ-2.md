---
title: RocketMQ部署日志
date: 2018-10-08 19:03:16
tags: RocketMQ
---

[RocketMQ搭建指南](https://rocketmq.apache.org/docs/quick-start/)

部署成功的日志信息如下
```
[root@karl-li-rocketmq rocketmq-all-4.3.0]# tail -f ~/logs/rocketmqlogs/broker.log

****************
****************
2018-10-08 19:20:12 INFO main - The broker[broker-a, 192.168.2.111:10911] boot success. serializeType=JSON and name server is localhost:9876
```

-------------------------------------------------------------------------------
### RocketMQ搭建排坑日志

问题1：由于nohup和&会使shell在后台运行， 所以导致报错信息get不到， 所以在部署过程中如果发现没有正常启动， 可以对命令进行替换， demo如下
```
nohup sh bin/mqnamesrv &
替换为
sh bin/mqnamesrv
```

问题2：搭建完毕后，客户端发送消息的时候出现如下报错

```
org.apache.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.2.111 :10909> failed
```

## 1. 多网卡问题（如果192.168.2.111不是Broker的IP）

由于RocketMQ的Broker端代码会获取IP返回给客户端， 当RocketMQ的服务端存在多网卡的时候返回了错误的IP给到客户端。

解决方案如下：

在`rocketmq-all-4.3.0/distribution/target/apache-rocketmq/conf/broker.conf`文件中指定nameServer和Broker的IP

```
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
namesrvAddr = 192.168.2.111:9876
brokerIP1 = 192.168.2.111
```

`PS：由于源代码没有任何的字符串trim操作，此处IP后面不要加任何空格或者Tab，否则会导致IP解析失败~~~~~血与泪的教训`


## 2. 网络环境问题

检测流程如下：
1. `ping 192.168.2.11`
2. `curl 192.168.2.111:9876`, `Empty reply from server`为正常， `Connection refused`说明有问题
```
➜  apache-rocketmq curl 192.168.2.111:9876
curl: (52) Empty reply from server
➜  apache-rocketmq curl 192.168.2.111:9871
curl: (7) Failed to connect to 192.168.2.111 port 9871: Connection refused
```



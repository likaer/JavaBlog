## 分布式基础理论

### 2PC(tow phase commit)两阶段提交。

所谓的两个阶段是指：第一阶段：准备阶段(投票阶段)和第二阶段：提交阶段（执行阶段）。

我们将提议的节点称为协调者(coordinator)，其他参与决议节点称为参与者(participants, 或cohorts)。

### 3PC

三阶段提交（Three-phase commit），是二阶段提交（2PC）的改进版本。

与两阶段提交不同的是，三阶段提交有两个改动点。

- 引入超时机制。同时在协调者和参与者中都引入超时机制。
- 在第一阶段和第二阶段中插入一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。

 也就是说，除了引入超时机制之外，3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有`CanCommit`、`PreCommit`、`DoCommit`三个阶段。

### CAP

[一致性](https://baike.baidu.com/item/一致性/9840083)（Consistency）、[可用性](https://baike.baidu.com/item/可用性/109628)（Availability）、[分区容错性](https://baike.baidu.com/item/分区容错性/23734073)（Partition tolerance）。CAP 原则指的是，这三个[要素](https://baike.baidu.com/item/要素/5261200)最多只能同时实现两点，不可能三者兼顾。

![img](images/cap.png)

**CA without P：**如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但放弃P的同时也就意味着放弃了系统的扩展性，也就是分布式节点受限，没办法部署子节点，这是违背分布式系统设计的初衷的。传统的关系型数据库RDBMS：Oracle、MySQL就是CA。

**CP without A：**如果不要求A（可用），相当于每个请求都需要在服务器之间保持强一致，而P（分区）会导致同步时间无限延长(也就是等待数据同步完才能正常访问服务)，一旦发生网络故障或者消息丢失等情况，就要牺牲用户的体验，等待所有数据全部一致了之后再让用户访问系统。设计成CP的系统其实不少，最典型的就是分布式数据库，如Redis、HBase等。对于这些分布式数据库来说，数据的一致性是最基本的要求，因为如果连这个标准都达不到，那么直接采用关系型数据库就好，没必要再浪费资源来部署分布式数据库。

 **AP wihtout C：**要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。典型的应用就如某米的抢购手机场景，可能前几秒你浏览商品的时候页面提示是有库存的，当你选择完商品准备下单的时候，系统提示你下单失败，商品已售完。这其实就是先在 A（可用性）方面保证系统可以正常的服务，然后在数据的一致性方面做了些牺牲，虽然多少会影响一些用户体验，但也不至于造成用户购物流程的严重阻塞。

### BASE

基本可用（Basically Available）软状态（Soft State）最终一致性（Eventually Consistent）

即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）

### Paxos

选举制算法，保证分布式系统数据的强一致性，由于需要多分区数据的一致性，

## 分布式锁

### Redis实现方案

```
基于Redisson实现方案

1. 使用Redis的Hash结构存储锁，设置30秒的超时时间
2. 支持锁重入，每上一次锁，value+1；通过ThreadId来判断锁持有
3. 使用LUA脚本，保证上锁逻辑和解锁逻辑的原子性
4. 使用WatchDog，每隔10s(1/3的ExpireTime)，进行一次锁超时续费，保证锁永不过期(TimerTask)
```

Redis满足CAP理论的AP理论，因为在集群部署方案下，Redis主从的数据是异步同步的

注：当主服务锁成功，然后挂掉，从服务又没同步到主服务的锁数据的时候，此时一致性就出错了

### ZooKeeper实现方案

```
1. 基于Zk创建临时顺序节点，序号最小的节点获得锁
2. 当最小的节点执行业务流程后需要删除节点，以释放锁
3. 每个临时顺序节点都监听上一个顺序节点，当上一个顺序节点已删除，则获得锁
```

ZooKeeper满足CAP理论的CP理论，因为在集群部署方案下，ZK遵循半数以上的数据需同步数据成功，并不利于高性能场景

Zookeeper需要安装奇数台(选举机制)

### MySQL实现方案

利用唯一索引实现



## 分布式事务的解决方案(Seata)

### AT模式

- 核心思想是基于SQL解析获取业务级UNDO LOG，并插入回滚日志表
- 事务全部提交成功则删除UNDO LOG
- 事务提交失败，则基于UNDO LOG回滚数据

该方案有性能瓶颈，不适用于所有业务场景

### TCC模式

TCC 模式，不依赖于底层数据资源的事务支持：

- 一阶段 try 行为：调用 **自定义** 的 prepare逻辑。
- 二阶段 commit 行为：调用 **自定义** 的 commit 逻辑。
- 二阶段 rollback 行为：调用 **自定义** 的 rollback 逻辑。

所谓 TCC 模式，是指支持把 **自定义** 的分支事务纳入到全局事务的管理中

### SAGA模式

Saga模式是SEATA提供的长事务解决方案，在Saga模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现

***SAGA相比较TCC少了prepare阶段, 且无法保证隔离性***

### XA模式(刚性事务)

XA 规范 是 X/Open 组织定义的分布式事务处理（DTP，Distributed Transaction Processing）标准

XA 规范 使用两阶段提交（2PC，Two-Phase Commit）来保证所有资源同时提交或回滚任何特定的事务

XA模式采用一个事务管理器来管理分布式事务，所有事务在执行前需要注册到事务管理器

- 事务发起者发起事务并在事务管理器注册事务ID
- 事务发起者调用其他事务参与者并传递事务ID
- 事务参与者注册参与事务ID的事务，并在事务执行完毕后通知事务管理者可以提交
- 事务发起者在事务执行完毕后告诉事务管理者提交事务，事务管理者同步提交事务给所有事务参与者

### 本地消息表(柔性事务)

- 对业务请求实现本地异步化，标记业务状态为执行中
- 通过重试解决网络问题，接口支持幂等
- 通过业务回滚方法解决消息消费失败的场景
- 本地消息表告警机制

### 基于消息中间件异步化

- 基于消息中间件异步化，将一个长事务分为短事务和柔性事务
- 通过重试机制保障柔性事务一定会执行
- 通过监控机制监控消费失败的消息

---

### Spring Cloud Nacos

- 配置中心
  - 动态管理和配置应用
  - 国际化语言配置
- 微服务管理
  - 服务注册与发现
  - 

## Dubbo

![/dev-guide/images/dubbo-relation.jpg](https://dubbo.apache.org/imgs/dev/dubbo-relation.jpg)

#### SPI加载机制(TODO)

SPI(Service Privoder Interface)是一种将接口和实现隔离，基于配置实现实例化的机制，举个例子

```
1. 制定统一的规范（比如 java.sql.Driver）

2. 服务提供商提供这个规范具体的实现，在自己jar包的META-INF/services/目录里创建一个以服务接口命名的文件，内容是实现类的全命名（比如：com.mysql.jdbc.Driver）。

3. 平台引入外部模块的时候，就能通过该jar包META-INF/services/目录下的配置文件找到该规范具体的实现类名，然后装载实例化，完成该模块的注入。

这个机制最大的优点就是无须在代码里指定，进而避免了代码污染，实现了模块的可拔插。

```



![/dev-guide/images/dubbo-framework.jpg](https://dubbo.apache.org/imgs/dev/dubbo-framework.jpg)

- **config 配置层**：对外配置接口，以 `ServiceConfig`, `ReferenceConfig` 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类
- **proxy 服务代理层**：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 `ServiceProxy` 为中心，扩展接口为 `ProxyFactory`
- **registry 注册中心层**：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 `RegistryFactory`, `Registry`, `RegistryService`
- **cluster 路由层**：封装多个提供者的路由及负载均衡，并桥接注册中心，以 `Invoker` 为中心，扩展接口为 `Cluster`, `Directory`, `Router`, `LoadBalance`
- **monitor 监控层**：RPC 调用次数和调用时间监控，以 `Statistics` 为中心，扩展接口为 `MonitorFactory`, `Monitor`, `MonitorService`
- **protocol 远程调用层**：封装 RPC 调用，以 `Invocation`, `Result` 为中心，扩展接口为 `Protocol`, `Invoker`, `Exporter`
- **exchange 信息交换层**：封装请求响应模式，同步转异步，以 `Request`, `Response` 为中心，扩展接口为 `Exchanger`, `ExchangeChannel`, `ExchangeClient`, `ExchangeServer`
- **transport 网络传输层**：抽象 mina 和 netty 为统一接口，以 `Message` 为中心，扩展接口为 `Channel`, `Transporter`, `Client`, `Server`, `Codec`
- **serialize 数据序列化层**：可复用的一些工具，扩展接口为 `Serialization`, `ObjectInput`, `ObjectOutput`, `ThreadPool`



#### Dubbo分层

Dubbo大致可以分为3层

- Business层，主要定义了Provider和Consumer基于接口的调用形式
- RPC层，主要定义了Provider和Consumer接口如何实现远程调用
  - proxy层对Consumer和Provider使用了代理的形式调用/提供实际的远程服务
  - Cluster提供了远程调用的负责均衡逻辑
  - Monitor提供了监控逻辑
  - protocol封装了RPC实际的调用方案，这一层方案依赖了Remoting层
- Remoting层，主要定义了数据传输方案
  - exchange，定义了客户端和服务端，通过管道进行数据传输
  - transport，真正传输数据，Netty在这一层工作
  - serialize，序列化

#### Dubbo流程

Dubbo流程大概可以分为4个阶段

- 初始化(发生在应用启动阶段)，基于Spring场景主要是针对配置的初始化，如解析duboo.xml，或者基于注解
- 服务暴露(发生在Provider应用启动阶段)，Provider将服务注册并暴露给客户端
- 服务引用(发生在Consumer应用启动阶段，当接口被Autowired的时候)，Consumer调用注册中心创建接口的远程引用Invoker
- 服务调用，Consumer调用远程引用Invoker，与Provider基于接口交互

### 

#### Dubbo初始化

1. dubbo实现了Spring的`BeanDefinitionParser`, 通过`DubboBeanDefinitionParser`和`AnnotationBeanDefinitionParser`解析dubbo的配置

2. Dubbo的DubboNamespaceHandler则将解析的结果注册到了Spring

   ```JAVA
   registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
           registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
           registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
           registerBeanDefinitionParser("config-center", new DubboBeanDefinitionParser(ConfigCenterBean.class, true));
           registerBeanDefinitionParser("metadata-report", new DubboBeanDefinitionParser(MetadataReportConfig.class, true));
           registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
           registerBeanDefinitionParser("metrics", new DubboBeanDefinitionParser(MetricsConfig.class, true));
           registerBeanDefinitionParser("ssl", new DubboBeanDefinitionParser(SslConfig.class, true));
           registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
           registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
           registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
           registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
           registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, true));
           registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
   ```

   

#### Dubbo服务暴露

1. 在Dubbo初始化完成后`ServiceBean`会通过实现afterPropertiesSet后触发ServiceBean的配置信息存储
2. Dubbo的ServiceBean同时实现ApplicationListener<ContextRefresh>，从而监听容器初始化完毕后，将服务注册到注册中心,并生成一个`Invoker`，记录接口，注册信息，代理的接口实现类方法
3. 底层注册完以后会创建`NettyServer`并监听端口



#### Dubbo服务引用

1. 在Spring对Service依赖的接口做Autowired的时候，Dubbo通过`ReferenceBean`获取需要Autowired列表
2. 基于注册中心获取URL，根据注册中心返回的信息创建`ProviderModel`, 同时订阅服务
3. 底层创建Netty客户端，创建`Invoker`代理并注入给Spring Bean

注：Consumer和Provider最终都被`ServiceRepository`所保存在`ConcurrentMap`中



#### Dubbo服务调用


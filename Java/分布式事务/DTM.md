# DTM

DTM（Distributed Transactions Manager）是一款开源的分布式事务管理器，解决跨数据库、跨服务、跨语言栈更新数据的一致性问题。

## 部署运维

### 角色
* `RM-资源管理器`：RM是一个应用服务，负责管理全局事务中的本地事务，他通常会连接到一个数据库，负责相关数据的修改、提交、回滚、补偿等操作。
* `AP-应用程序`：AP是一个应用服务，负责全局事务的编排，他会注册全局事务，注册子事务，调用RM接口。
* `TM-事务管理器`：TM就是DTM服务，负责全局事务的管理，每个全局事务都注册到TM，每个事务分支也注册到TM。TM会协调所有的RM，将同一个全局事务的不同分支，全部提交或全部回滚。

> 参考链接 - [DTM架构](https://dtm.pub/practice/arch.html)

### 1. RM 子事务屏障表
在**业务数据库**中创建`子事务屏障表`，详见: [barrier SQL文件](https://github.com/dtm-labs/dtm/blob/main/sqls/)

### 2. `DTM服务器`使用`数据库存储`配置
创建`全局事务信息`表，详见: [storage SQL文件](https://github.com/dtm-labs/dtm/blob/main/sqls/)

> 注：一定要使用主库。一方面dtm是写多读少，读写分离对于dtm的负载分摊有限；另一方面dtm对数据的一致性要求较高，从库延时会导致各种问题。

### 3. DTM配置
DTM支持环境变量和文件两种配置，如果同时有环境变量和文件，那么**配置文件的优先级高**。

#### 环境变量
所有可配置的选项参考: [YML样板配置文件](https://github.com/dtm-labs/dtm/blob/main/conf.sample.yml) ，对于每个配置文件里的配置，都可以通过环境变量设置，对应规则如下：
```
MicroService.EndPoint => MICRO_SERVICE_END_POINT
```

#### YAML文件配置
DTM支持YML配置文件，详细配置项参考[YML样板配置文件](https://github.com/dtm-labs/dtm/blob/main/conf.sample.yml)

服务启动时指定配置文件：`dtm -c ./conf.sample.yml`

```yaml
#####################################################################
### dtm can be run without any config.
### all config in this file is optional. the default value is as specified in each line
### all configs can be specified from env. for example:
### MicroService.EndPoint => MICRO_SERVICE_END_POINT
#####################################################################

# Store: # specify which engine to store trans status
#   Driver: 'mysql'
#   Host: 'localhost'
#   User: 'root'
#   Password: '123456'
#   Port: 3306
#   Driver: 'boltdb' # default store engine

#   Driver: 'redis'
#   Host: 'localhost'
#   User: ''
#   Password: ''
#   Port: 6379

#   Driver: 'postgres'
#   Host: 'localhost'
#   User: 'postgres'
#   Password: 'mysecretpassword'
#   Port: '5432'

### following config is for only Driver postgres/mysql
#   MaxOpenConns: 500
#   MaxIdleConns: 500
#   ConnMaxLifeTime: 5 # default value is 5 (minutes)
#   TransGlobalTable: 'dtm.trans_global'
#   TransBranchOpTable: 'dtm.trans_branch_op'

### flollowing config is only for some Driver
#   DataExpire: 604800 # Trans data will expire in 7 days. only for redis/boltdb.
#   FinishedDataExpire: 86400 # finished Trans data will expire in 1 days. only for redis.
#   RedisPrefix: '{a}' # default value is '{a}'. Redis storage prefix. store data to only one slot in cluster

# MicroService: # grpc based microservice config
#   Driver: 'dtm-driver-gozero' # name of the driver to handle register/discover
#   Target: 'etcd://localhost:2379/dtmservice' # register dtm server to this url
#   EndPoint: 'localhost:36790'

# HttpMicroService: # http based microservice config
#   Driver: 'dtm-driver-http' # name of the driver to handle register/discover
#   RegistryType: 'nacos'
#   RegistryAddress: '127.0.0.1:8848,127.0.0.1:8848'
#   RegistryOptions: '{"UserName":"nacos","Password":"nacos","NotLoadCacheAtStart":true}'
#   Target: '{"ServiceName":"dtmService","Enable":true,"Healthy":true,"Weight":10}' # target and options
#   EndPoint: '127.0.0.1:36789'

### the unit of following configurations is second
# TransCronInterval: 3 # the interval to poll unfinished global transaction for every dtm process
# TimeoutToFail: 35 # timeout for XA, TCC to fail. saga's timeout default to infinite, which can be overwritten in saga options
# RetryInterval: 10 # the subtrans branch will be retried after this interval
# RequestTimeout: 3 # the timeout of HTTP/gRPC request in dtm

# LogLevel: 'info'              # default: info. can be debug|info|warn|error
# Log:
#    Outputs: 'stderr'           # default: stderr, split by ",", you can append files to Outputs if need. example:'stderr,/tmp/test.log'
#    RotationEnable: 0           # default: 0
#    RotationConfigJSON: '{}'    # example: '{"maxsize": 100, "maxage": 0, "maxbackups": 0, "localtime": false, "compress": false}'
#
# HttpPort: 36789
# GrpcPort: 36790
# JsonRpcPort: 36791

### advanced options
# UpdateBranchAsyncGoroutineNum: 1 # num of async goroutine to update branch status
```

##### DTM 集成 MySQL
```yaml
# specify which engine to store trans status
Store:
  Driver: 'mysql'
  Host: 'localhost'
  Port: 3306
  User: 'root'
  Password: '123456'
  #MaxOpenConns: 500
  #MaxIdleConns: 500
  #ConnMaxLifeTime: 5 # default value is 5 (minutes)
```

##### DTM 集成 Redis
```yaml
# specify which engine to store trans status
Store:
  Driver: 'redis'
  Host: 'localhost'
  Port: 6379
  User: ''
  Password: ''
  #DataExpire: 604800 # Trans data will expire in 7 days. only for redis/boltdb.
  #FinishedDataExpire: 86400 # finished Trans data will expire in 1 days. only for redis.
  #RedisPrefix: '{a}' # default value is '{a}'. Redis storage prefix. store data to only one slot in cluster
```


DTM默认监听端口：
* HttpPort：36789
* GrpcPort：36790
* JsonRpcPort: 36791

### 4. 部署DTM - [链接](https://dtm.pub/deploy/deploy.html)
* 可执行文件部署
* Docker部署
* Kubernetes部署
* Helm部署

### 5. 运维 - [链接](https://dtm.pub/deploy/maintain.html)
* **报警** - 正常情况下，全局事务会很快结束，结果在dtm.trans_global的status记录为succeed/failed，即使出现临时的网络问题等，通常也会在一两次重试之后，最终结束。
如果重试次数超过3，通常意味着异常情况，最好能够监测起来。
* **立即触发全局事务重试** - dtm对每个事务的重试是指数退避策略，具体为间隔是每失败一次，间隔加倍，避免过多的重试，导致系统负载异常上升。
如果您经过长时间的的宕机，因指数退避算法导致要很久才会重试。如果您想要手动触发立即重试，您可以手动把相应事务的
next_cron_time(Redis存储引擎的该功能还在开发中)修改为当前时间，就会在数秒内被定时轮询，事务就会继续往前执行。


## 架构与设计

### 分布式事务理论 - [链接](https://dtm.pub/practice/theory.html)

#### 1. 事务

把多条语句作为一个整体进行操作的功能，被称为数据库事务。数据库事务可以确保该事务范围内的所有操作都可以全部成功或者全部失败。

事务具有 4 个属性：原子性、一致性、隔离性、持久性。这四个属性通常称为 ACID 特性。

* `Atomicity（原子性）`：一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复到事务开始前的状态，就像这个事务从来没有执行过一样。
* `Consistency（一致性）`：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。完整性包括外键约束、应用定义的等约束不会被破坏。
* `Isolation（隔离性）`：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。
* `Durability（持久性）`：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

主流的数据库例如Mysql、Postgres等，都支持ACID事务，其内部会采用`MVCC`（多版本并发控制）技术，实现高性能、高并发的本地事务。MVCC技术是数据库的技术基石，非常重要，但是内容很多，这里就不再展开细说，有兴趣的同学，可以自行查找相关资料学习。

#### 2. 分布式理论

分布式事务涉及多个节点，是一个典型的分布式系统，与单机系统有非常大的差别。一个分布式系统最多只能同时满足`一致性（Consistency）`、
`可用性（Availability）`和`分区容错性（Partition tolerance）`这三项中的两项，这被称为`CAP理论`。

##### C 一致性
分布式系统中，数据一般会存在不同节点的副本中，如果对第一个节点的数据成功进行了更新操作，而第二个节点上的数据却没有得到相应更新，这时候读取第二个节点的数据依然是更新前的数据，即脏数据，这就是分布式系统数据不一致的情况。

在分布式系统中，如果能够做到针对一个数据项的更新操作执行成功后，所有的用户都能读取到最新的值，那么这样的系统就被认为具有强一致性（或严格的一致性）。

请注意CAP中的一致性和ACID中的一致性，虽然单词相同，但实际含义不同，请注意区分

##### A 可用性
在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。

在现代的互联网应用中，如果因为服务器宕机等问题，导致服务长期不可用，是不可接受的

##### P 分区容错性
以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在 C 和 A 之间做出选择。

提高分区容忍性的办法就是一个数据项复制到多个节点上，那么出现分区之后，这一数据项仍然能在其他区中读取，容忍性就提高了。然而，把数据复制到多个节点，就会带来一致性的问题，就是多个节点上面的数据可能是不一致的。

##### 面临的问题
对于多数大型互联网应用的场景，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到N个9，即保证P和A，舍弃C。

##### BASE理论
`BASE`是`Basically Available`（基本可用）、`Soft state`（软状态）和`Eventually consistent`（最终一致性）三个短语的简写，BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。接下来我们着重对BASE中的三要素进行详细讲解。

* 基本可用是指分布式系统在出现不可预知故障的时候，允许损失部分可用性——但请注意，这绝不等价于系统不可用。
* 弱状态也称为软状态，和硬状态相对，是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据同步的过程存在延时。
* 最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性

总的来说，BASE理论面向的是大型高可用可扩展的分布式系统，提出通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。

许多的NoSQL是按照BASE理论进行设计的，典型的例子包括：Dynamo、Cassandra、CouchDB。

#### 3. 分布式事务

#### 4. NewSQL的分布式事务

#### 5. 跨服务跨库的分布式事务

#### 6. 无法强一致

#### 7. 理论上的强一致性

#### 8. 最终一致性

### DTM架构 - [链接](https://dtm.pub/practice/arch.html)

### 异常与子事务屏障 - [链接](https://dtm.pub/practice/barrier.html)

#### 1. NPC的挑战
分布式系统最大的敌人可能就是NPC了，在这里它是Network Delay, Process Pause, Clock Drift的首字母缩写。我们先看看具体的NPC问题是什么：

* `Network Delay`，网络延迟。虽然网络在多数情况下工作的还可以，虽然TCP保证传输顺序和不会丢失，但它无法消除网络延迟问题。
* `Process Pause`，进程暂停。有很多种原因可以导致进程暂停：比如编程语言中的GC（垃圾回收机制）会暂停所有正在运行的线程；再比如，我们有时会暂停云服务器，从而可以在不重启的情况下将云服务器从一台主机迁移到另一台主机。我们无法确定性预测进程暂停的时长，你以为持续几百毫秒已经很长了，但实际上持续数分钟之久进程暂停并不罕见。
* `Clock Drift`，时钟漂移。现实生活中我们通常认为时间是平稳流逝，单调递增的，但在计算机中不是。计算机使用时钟硬件计时，通常是石英钟，计时精度有限，同时受机器温度影响。为了在一定程度上同步网络上多个机器之间的时间，通常使用NTP协议将本地设备的时间与专门的时间服务器对齐，这样做的一个直接结果是设备的本地时间可能会突然向前或向后跳跃。

分布式事务既然是分布式的系统，自然也有NPC问题。因为没有涉及时间戳，带来的困扰主要是NP。

#### 2. 异常分类
我们以分布式事务中的TCC作为例子，看看NP带来的影响。

一般情况下，一个TCC回滚时的执行顺序是，先执行完Try，再执行Cancel，但是由于N，则有可能Try的网络延迟大，导致先执行Cancel，再执行Try。

这种情况就引入了分布式事务中的两个难题：

* `空补偿`： Cancel执行时，Try未执行，事务分支的Cancel操作需要判断出Try未执行，这时需要忽略Cancel中的业务数据更新，直接返回
* `悬挂`： Try执行时，Cancel已执行完成，事务分支的Try操作需要判断出Cancel已执行，这时需要忽略Try中的业务数据更新，直接返回

分布式事务还有一类需要处理的常见问题，就是重复请求

* `幂等`： 由于任何一个请求都可能出现网络异常，出现重复请求，所有的分布式事务分支操作，都需要保证幂等性

因为空补偿、悬挂、重复请求都跟NP有关，我们把他们统称为子事务乱序问题。在业务处理中，需要小心处理好这三种问题，否则会出现错误数据。

#### 3. 网络异常的时序图

#### 4. 现有方案的问题

#### 5. 子事务屏障

#### 6. 原理

### 最终成功 - [链接](https://dtm.pub/practice/must-succeed.html)

### 如何选择事务模式

#### 特性对比
* `二阶段消息模式`: 适合不需要回滚的场景
* `saga模式`: 适合需要回滚的场景
* `tcc事务模式`: 适合一致性要求较高的场景
* `xa事务模式`: 适合并发要求不高，没有数据库行锁争抢的场景

#### 各模式详解
* [SAGA事务模式](https://dtm.pub/practice/saga.html)
* [二阶段消息](https://dtm.pub/practice/msg.html)
* [TCC事务模式](https://dtm.pub/practice/tcc.html)
* [XA事务模式](https://dtm.pub/practice/xa.html)
* [其他事务模式](https://dtm.pub/practice/other.html)
  * 本地消息表
  * 事务消息
  * 最大努力通知
  * AT事务模式
* [AT vs XA](https://dtm.pub/practice/at.html)

#### 应用场景
* `秒杀场景`：当秒杀访问量很大时，多数系统都会选择在redis中扣减库存，扣减成功后再创建订单。这种场景下没有回滚，适合二阶段消息。
二阶段消息还能够保证出现进程崩溃的情况下，扣减的库存量与创建的订单量是完全相等的，详情参见[秒杀应用](https://dtm.pub/app/flash.html)
* `缓存管理场景`：使用redis缓存来提供数据，降低数据库的压力是非常常见的场景。通常情况下，会先更新DB，在更新缓存，不涉及回滚，
适合二阶段消息。详情参见[缓存应用](https://dtm.pub/app/cache.html)
* 假设您有一个订单业务，需要保证您的订单创建，库存扣减，优惠券扣减是同时成功或同时失败的。那么它适合Saga，因为Saga能够支持回滚，
是支持回滚的分布式事务里，最易用的。详情参见[订单应用](https://dtm.pub/app/order.html)
* 假设您有一个类似资金转账的业务，对一致性要求较高，不允许在转账失败的情况下，让用户看到中间的余额变动，这种情况适合TCC，
它能够灵活的控制整个全局事务的数据可见性。详情参见[TCC事务](https://segmentfault.com/a/1190000040331793)
* 假设您的业务，对并发性要求不高，也不会出现多个请求争用同一行数据（例如不会出现扣减同一个商品库存的情况），那么可以选用`XA`。

#### 典型应用
* [概述](https://dtm.pub/app/intro.html)
* [订单系统](https://dtm.pub/app/order.html)
* [秒杀](https://dtm.pub/app/flash.html)
* [缓存一致性](https://dtm.pub/app/cache.html)


## 应用接入

### SDK - [链接](https://dtm.pub/ref/sdk.html)

#### 1. 支持的语言
#### 2. 支持的数据库
> dtm的SDK中，提供了子事务屏障功能，也提供了XA相关的支持，这部分的支持是与具体的数据库相关的
#### 3. ORM 对接

### 事务配置项 - [链接](https://dtm.pub/ref/options.html)
1. 等待事务结果
2. 超时
3. 重试时间
4. 重试时间
5. 自定义header - 可用于鉴权功能

### 存储引擎 - [链接](https://dtm.pub/ref/store.html)
1. 关系数据库
2. Redis
3. boltdb

### 支持的协议
1. HTTP
2. gRPC
3. 基于gRPC的微服务协议

### 单服务多数据源 - [链接](https://dtm.pub/ref/feature.html#%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90)

### HTTP API - [链接](https://dtm.pub/ref/http.html)
1. newGid 获取Gid
2. prepare 准备事务
3. submit 提交事务
4. abort 回滚事务
5. registerBranch
6. query 查询事务
7. 事务选项


## 文档

* [DTM vs SEATA](https://dtm.pub/other/opensource.html)



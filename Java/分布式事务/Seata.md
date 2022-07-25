# Seata

Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。
Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。


## 术语

### TC (Transaction Coordinator) - 事务协调者
维护全局和分支事务的状态，驱动全局事务提交或回滚。

### TM (Transaction Manager) - 事务管理器
定义全局事务的范围：开始全局事务、提交或回滚全局事务。

### RM (Resource Manager) - 资源管理器
管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

## 数据源支持

### AT（Automatic Transaction）模式
AT模式支持的数据库有：MySQL、Oracle、PostgreSQL、 TiDB、MariaDB。
> [Seata AT 模式](https://seata.io/zh-cn/docs/dev/mode/at-mode.html)

### TCC（Try-Confirm-Cancel）模式
TCC模式不依赖数据源(1.4.2版本及之前)，1.4.2版本之后增加了TCC防悬挂措施，需要数据源支持。
* [Seata TCC 模式](https://seata.io/zh-cn/docs/dev/mode/tcc-mode.html)
* [Seata TCC 示例](https://github.com/seata/seata-samples/tree/master/tcc)

### Saga模式
Saga模式不依赖数据源。

* [Seata Saga 模式](https://seata.io/zh-cn/docs/user/saga.html)
* [Seata Saga 示例](https://github.com/seata/seata-samples/tree/master/saga)

> 配置客户端参数client.rm.report.success.enable=false，可以在当分支事务执行成功时不上报分支状态到server，从而提升性能。

> 注：**当上一个分支事务的状态还没有上报的时候，下一个分支事务已注册，可以认为上一个实际已成功**

### XA（eXtended Architecture）模式
XA模式只支持实现了XA协议的数据库。Seata支持MySQL、Oracle、PostgreSQL和MariaDB。
* [Seata XA 模式](https://seata.io/zh-cn/docs/dev/mode/xa-mode.html)
* [Seata XA 示例](https://github.com/seata/seata-samples/tree/master/seata-xa)

1.4.2版本XA模式BUG：
* [xa模式获取channel导致的资源悬挂问题](https://github.com/seata/seata/issues/4138)
  * [bugfix: xa resource suspension](https://github.com/seata/seata/pull/4228)
* [XA模式资源悬挂问题](https://github.com/seata/seata/issues/4073)
  * [bugfix: prevents XA mode resource suspension](https://github.com/seata/seata/pull/4074)

## 事务状态

### 全局事务状态表
以db模式举例，global_table是seata的全局事务表。你可以通过观察global_table表中status字段知悉全局事务处于哪个状态

|状态| 	代码 |	备注|
|:--------------:|:---:|:-----------------:|
|全局事务开始（Begin）| 	1	 |此状态可以接受新的分支事务注册|
|全局事务提交中（Committing）| 	2	 |这个状态会随时改变|
|全局事务提交重试（CommitRetry）| 	3	 |在提交异常被解决后尝试重试提交|
|全局事务回滚中（Rollbacking）| 	4	 |正在重新回滚全局事务|
|全局事务回滚重试中（RollbackRetrying）| 	5	 |在全局回滚异常被解决后尝试事务重试回滚中|
|全局事务超时回滚中（TimeoutRollbacking）| 	6	 |全局事务超时回滚中|
|全局事务超时回滚重试中（TimeoutRollbackRetrying）| 	7	 |全局事务超时回滚重试中|
|异步提交中（AsyncCommitting）| 	8	 |异步提交中|
|二阶段已提交（Committed）| 	9	 |二阶段已提交，此状态后全局事务状态不会再改变|
|二阶段提交失败（CommitFailed）|	10	 |二阶段提交失败|
|二阶段决议全局回滚（Rollbacked）|	11	 |二阶段决议全局回滚|
|二阶段全局回滚失败（RollbackFailed）|	12	 |二阶段全局回滚失败|
|二阶段超时回滚（TimeoutRollbacked）|	13	 |二阶段超时回滚|
|二阶段超时回滚失败（TimeoutRollbackFailed）|	14	 |二阶段超时回滚失败|
|全局事务结束（Finished）|	15	 |全局事务结束|
|二阶段提交超时（CommitRetryTimeout）|	16	 |二阶段提交因超过重试时间限制导致失败|
|二阶段回滚超时（RollbackRetryTimeout）|	17	 |二阶段回滚因超过重试时间限制导致失败|
|未知状态（UnKnown）| 	0	 |未知状态|

**全局事务超时回滚中（TimeoutRollbacking）**

怎么发生的？
1. 当某个seata全局事务执行过程中，无法完成业务。
2. TC中的一个定时任务（专门用来寻找已超时的全局事务），发现该全局事务未回滚完成，就会将此全局事务改为全局事务超时回滚中（TimeoutRollbacking），开始回滚，直到回滚完毕后删除global_table数据。

建议：当你发现全局事务处于该状态，请排查为何业务无法在限定时间内完成事务。若确实无法完成，应调大全局事务超时时间。（如排查一切正常，请检查tc集群时区与数据库是否一致，若不一致请改为一致）。

### 分支事务状态表

|状态|	代码	|备注|
|:--------------:|:---:|:-----------------:|
|分支事务注册（Registered）|	1	|向TC注册分支事务|
|分支事务一阶段完成（PhaseOne_Done）|	2	|分支事务一阶段业务逻辑完成|
|分支事务一阶段失败（PhaseOne_Failed）|	3	|分支事务一阶段业务逻辑失败|
|分支事务一阶段超时（PhaseOne_Timeout）|	4	|分支事务一阶段处理超时|
|分支事务二阶段已提交（PhaseTwo_Committed）|	5	|分支事务二阶段提交|
|分支事务二阶段提交失败重试（PhaseTwo_CommitFailed_Retryable）|	6	|分支事务二阶段提交失败重试|
|分支事务二阶段提交失败不重试（PhaseTwo_CommitFailed_Unretryable）|	7	|分支事务二阶段提交失败不重试|
|分支事务二阶段已回滚（PhaseTwo_Rollbacked）|	8	|分支事务二阶段已回滚|
|分支事务二阶段回滚失败重试（PhaseTwo_RollbackFailed_Retryable）|	9	|分支事务二阶段回滚失败重试|
|分支事务二阶段回滚失败不重试（PhaseTwo_RollbackFailed_Unretryable）|	10	|二阶段提交失败|
|未知状态（UnKnown）|	0	|未知状态|


## 部署

* [部署指南](https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)
  * 资源目录介绍
  * 启动Server
  * 业务系统集成Client
* [参数配置](https://seata.io/zh-cn/docs/user/configurations.html)
* 部署方式
  * [直接部署](https://seata.io/zh-cn/docs/ops/deploy-server.html)
  * [Docker 部署](https://seata.io/zh-cn/docs/ops/deploy-by-docker.html)
  * [docker-compose 部署](https://seata.io/zh-cn/docs/ops/deploy-by-docker-compose.html)
  * [Kubernetes 部署](https://seata.io/zh-cn/docs/ops/deploy-by-kubernetes.html)
  * [Helm 部署](https://seata.io/zh-cn/docs/ops/deploy-by-helm.html)
* [Seata 高可用部署](https://seata.io/zh-cn/docs/ops/deploy-ha.html)

## FAQ

> [FAQ官方链接](https://seata.io/zh-cn/docs/overview/faq.html)

### Q: undo_log表log_status=1的记录是做什么用的？
A:
* 场景 ： 分支事务a注册TC后，a的本地事务提交前发生了全局事务回滚
* 后果 ： 全局事务回滚成功，a资源被占用掉，产生了资源悬挂问题
* 防悬挂措施： a回滚时发现回滚undo还未插入，则插入一条log_status=1的undo记录，a本地事务（业务写操作sql和对应undo为一个本地事务）提交时会因为undo表唯一索引冲突而提交失败。

### Q: 怎么使用Seata框架，来保证事务的隔离性？
A: 因seata一阶段本地事务已提交，为防止其他事务脏读脏写需要加强隔离。
1. 脏读 select语句加for update，代理方法增加@GlobalLock+@Transactional或@GlobalTransaction
2. 脏写 必须使用@GlobalTransaction
> 注：如果你查询的业务的接口没有GlobalTransactional 包裹，也就是这个方法上压根没有分布式事务的需求，这时你可以在方法上标注
@GlobalLock+@Transactional 注解，并且在查询语句上加 for update。 如果你查询的接口在事务链路上外层有GlobalTransactional注解，
那么你查询的语句只要加for update就行。设计这个注解的原因是在没有这个注解之前，需要查询分布式事务读已提交的数据，但业务本身不需要分布式事务。 
若使用GlobalTransactional注解就会增加一些没用的额外的rpc开销比如begin 返回xid，提交事务等。GlobalLock简化了rpc过程，使其做到更高的性能。

### Q: 脏数据回滚失败如何处理?
A:
1. 脏数据需手动处理，根据日志提示修正数据或者将对应undo删除（可自定义实现FailureHandler做邮件通知或其他）
2. 关闭回滚时undo镜像校验，不推荐该方案。
> 注：建议事前做好隔离保证无脏数据

### Q: 为什么分支事务注册时, 全局事务状态不是begin?
A:
* 异常：Could not register branch into global session xid = status = Rollbacked（还有Rollbacking、AsyncCommitting等等二阶段状态） while expecting Begin
* 描述：分支事务注册时，全局事务状态需是一阶段状态begin，非begin不允许注册。属于seata框架层面正常的处理，用户可以从自身业务层面解决。
* 出现场景（可继续补充）
>  1. 分支事务是异步，全局事务无法感知它的执行进度，全局事务已进入二阶段，该异步分支才来注册
>  2. 服务a rpc 服务b超时（dubbo、feign等默认1秒超时），a上抛异常给tm，tm通知tc回滚，但是b还是收到了请求（网络延迟或rpc框架重试），然后去tc注册时发现全局事务已在回滚
>  3. tc感知全局事务超时(@GlobalTransactional(timeoutMills = 默认60秒))，主动变更状态并通知各分支事务回滚，此时有新的分支事务来注册

### Q: Nacos 作为 Seata 配置中心时，项目启动报错找不到服务。如何排查，如何处理?
A: [参考FAQ文档](https://seata.io/zh-cn/docs/overview/faq.html)

### Q: Eureka做注册中心，TC高可用时，如何在TC端覆盖Eureka属性?
A: [参考FAQ文档](https://seata.io/zh-cn/docs/overview/faq.html)

### Q: 为什么mybatis没有返回自增ID?
A: 
1. 方案1.需要修改mybatis的配置: 在@Options(useGeneratedKeys = true, keyProperty = "id")或者在xml中指定useGeneratedKeys 和 keyProperty属性
2. 方案2.删除undo_log表的id字段

### Q: io.seata.codec.protobuf.generated不存在，导致seata server启动不了?
A: [参考FAQ文档](https://seata.io/zh-cn/docs/overview/faq.html)

### Q: TC如何使用mysql8?
A: 
1. 修改file.conf的驱动配置store.db.driver-class-name; 
2. lib目录下删除mysql5驱动,添加mysql8驱动
> ps: oracle同理;1.2.0支持mysql驱动多版本隔离，无需再添加驱动

### Q: 支持多主键?
A: 暂时只支持mysql，其他类型数据库建议先建一列自增id主键，原复合主键改为唯一键来规避下

### Q: 使用HikariDataSource报错如何解决?
A:
* 异常1:ClassCastException: com.sun.proxy.$Proxy153 cannot be cast to com.zaxxer.hikari.HikariDataSource
* 原因: 自动代理时，实例类型转换错误，注入的是$Proxy153实例，不是HikariDataSource的本身或子类实例。
* 解决: seata自动代理数据源功能使用jdk proxy, 对DataSource进行代理，生成的代理类 extends Proxy implements DataSource, 接收方可改成DataSource接收实现。
1.1.0将同时支持jdk proxy和cglib，届时该问题还可切换cglib解决。

### Q: 是否可以不使用conf类型配置文件，直接将配置写入application.properties?
A: 目前seata-all是需要使用conf类型配置文件，后续会支持properties和yml类型文件。当前可以在项目中依赖`seata-spring-boot-starter`，
然后将配置项写入到application .properties 这样可以不使用conf类型文件。

### Q: 如何自己修改源码后打包seata-server?
A:
1. 删除 distribution 模块的bin、conf和lib目录。
2. ./mvnw clean install -DskipTests=true(Mac,Linux) 或 mvnw.cmd clean install -DskipTests=true(Win) -P release-seata。
3. 在 distribution 模块的 target 目录下解压相应的压缩包即可。
4. seata-1.5之后(最新develop分支)的打包命令：mvn -Prelease-seata -Dmaven.test.skip=true clean install -U

### Q: Seata 支持哪些 RPC 框架?
A:
1. AT 模式支持Dubbo、Spring Cloud、Motan、gRPC 和 sofa-RPC。
2. TCC 模式支持Dubbo、Spring Cloud和sofa-RPC。

### Q: java.lang.NoSuchMethodError: com.alibaba.druid.sql.ast.statement .SQLSelect.getFirstQueueBlockLcom/alibaba/druid/sql/ast/statement/SQLSelectQueryBlock;
A: 需要将druid的依赖版本升级至1.1.12+ 版本，Seata内部默认依赖的版本是1.1.12（provided）。

### Q: apache-dubbo 2.7.0出现NoSuchMethodError ?
A: [参考FAQ文档](https://seata.io/zh-cn/docs/overview/faq.html)

### Q: 使用 AT 模式需要的注意事项有哪些 ？
A:
1. 必须使用代理数据源，有 3 种形式可以代理数据源：
   * 依赖 seata-spring-boot-starter 时，自动代理数据源，无需额外处理。
   * 依赖 seata-all 时，使用 @EnableAutoDataSourceProxy (since 1.1.0) 注解，注解参数可选择 jdk 代理或者 cglib 代理。
   * 依赖 seata-all 时，也可以手动使用 DatasourceProxy 来包装 DataSource。
2. 配置 GlobalTransactionScanner，使用 seata-all 时需要手动配置，使用`seata-spring-boot-starter`时无需额外处理。
3. 业务表中必须包含单列主键，若存在复合主键，请参考问题 `支持多主键?` 。
4. 每个业务库中必须包含 undo_log 表，若与分库分表组件联用，分库不分表。
5. 跨微服务链路的事务需要对相应 RPC 框架支持，目前 seata-all 中已经支持：Apache Dubbo、Alibaba Dubbo、sofa-RPC、Motan、gRpc、httpClient，对于 Spring Cloud 的支持，请大家引用 spring-cloud-alibaba-seata。其他自研框架、异步模型、消息消费事务模型请结合 API 自行支持。
6. 目前AT模式支持的数据库有：MySQL、Oracle、PostgreSQL和 TiDB。
7. 使用注解开启分布式事务时，若默认服务 provider 端加入 consumer 端的事务，provider 可不标注注解。但是，provider 同样需要相应的依赖和配置，仅可省略注解。
8. 使用注解开启分布式事务时，若要求事务回滚，必须将异常抛出到事务的发起方，被事务发起方的 @GlobalTransactional 注解感知到。provide 直接抛出异常 或 定义错误码由 consumer 判断再抛出异常。

### Q: AT 模式和 Spring @Transactional 注解连用时需要注意什么 ？
A: `@Transactional` 可与 `DataSourceTransactionManager` 和 `JTATransactionManager` 连用分别表示本地事务和XA分布式事务，大家常用的是与本地事务结合。
当与本地事务结合时，`@Transactional`和`@GlobalTransaction`连用，`@Transactional` 只能位于标注在`@GlobalTransaction`的**同一方法层次**或者
位于`@GlobalTransaction` 标注方法的**内层**。这里分布式事务的概念要大于本地事务，若将 `@Transactional` 标注在外层会导致分布式事务空提交，
当@Transactional 对应的 connection 提交时会报全局事务正在提交或者全局事务的xid不存在。

### Q: Spring boot 1.5.x 出现 jackson 相关 NoClassDefFoundException ？

> Caused by: java.lang.NoClassDefFoundError: Could not initialize class com.fasterxml.jackson.databind.ObjectMapper

目前发现在 Spring Boot 1.5.x 版本中原始引入的 jackson 版本过低，会导致 Seata 依赖 jackson 的新特性找不到，Seata 要求 
jackson 版本2.9.9+，但是使用 jackson 2.9.9+ 版本会导致Spring Boot中使用的jackson API找不到，也就是jackson本身的向前兼容性存在问题。
因此,建议大家将Seata的序列化方式切换到非 jackson 序列化方式，比如 kryo，配置项为client.undo.logSerialization = "kryo"

### Q: SpringCloud xid无法传递 ？
A:
1. 首先确保你引入了`spring-cloud-starter-alibaba-seata`的依赖.
2. 如果xid还无法传递,请确认你是否实现了`WebMvcConfigurer`,如果是,请参考`com.alibaba.cloud.seata.web.SeataHandlerInterceptorConfiguration#addInterceptors`的方法. 
把`SeataHandlerInterceptor`加入到你的拦截链路中.

### Q: 使用mybatis-plus 动态数据源组件后undolog无法删除 ？
A: `dynamic-datasource-spring-boot-starter` 组件内部开启seata后会自动使用DataSourceProxy来包装DataSource,所以需要以下方式来保持兼容
1. 如果你引入的是seata-all,请不要使用@EnableAutoDataSourceProxy注解.
2. 如果你引入的是seata-spring-boot-starter 请关闭自动代理
```yaml
seata:
  enable-auto-data-source-proxy: false
```

### Q: Could not found global transaction xid = %s, may be has finished.
A: 举例说明：
```
@GlobalTransactional(timeout=60000) public void A（）{

 call remoting B();//远程调用B服务 local DB operation;

}

public void B(){

}
```
可能原因：
1. A 执行的总体时间超过了60000ms，导致全局事务发起了全局回滚，此时A或B方法继续执行DB操作，校验全局事务状态，发现全局事务已经回滚。
2. B服务执行超出其设定的readTimeout 返回异常给A并将异常抛出导致全局事务回滚，此时B服务执行DB操作时，校验全局事务状态，发现全局事务已经回滚。
3. tc集群节点时间不一致。

影响：出现这种情况时，数据会整体回滚至A方法执行前的数据的初态，从数据一致性的视角上看，数据是整体一致的。
> 除了上述情况，如果引用的是seata-spring-boot-starter的话，产生这个错误的原因也可能是因为一个bug，目前在1.5版本进行了修复，具体可以参考[issues4020](https://github.com/seata/seata/issues/4020)。

### Q: TC报这个错：An exceptionCaught() event was fired, and it reached at the tail of the pipeline. It usually means the last handler in the pipeline did not handle the exception是什么原因？
A：这个错误是由非正常Seata客户端建立连接引起（如通过http访问Seata server的端口，云服务器的端口扫描等）。这种连接没有发送注册信息，被认为是无用连接，该异常可以忽视。

### Q: 数据库开启自动更新时间戳导致脏数据无法回滚？
A: 由于业务提交，seata记录当前镜像后，数据库又进行了一次时间戳的更新，导致镜像校验不通过。
* 方案1: 关闭数据库的时间戳自动更新。数据的时间戳更新，如修改、创建时间由代码层面去维护，比如MybatisPlus就能做自动填充。
* 方案2: update语句别把没更新的字段也放入更新语句。

### Q: 还没到全局事务超时时间就出现了timeoutrollcking?
A: 有可能是多tc时区不一致导致的，建议将多个tc时区与db模式数据库时区保持一致统一。

### Q: Seata现阶段支持的分库分表解决方案？
A: 现阶段只支持ShardingSphere。关于分库分表与Seata兼容的问题，Seata支持某一个分库分表方案是需要分库分表框架团队来提供集成兼容方案，
而不是Seata提供。目前Seata正与各分库分表框架团队进行沟通来商讨集成兼容方案。

### Q: Seata 使用注册中心注册的地址有什么限制？
A: Seata 注册中心不能注册 `0.0.0.0` 或 `127.0.0.1` 的地址，当自动注册为上述地址时可以通过启动参数 `-h` 或容器环境变量`SEATA_IP`来指定。
当和业务服务处于不同的网络时注册地址可以指定为 `NAT_IP`或`公网IP`，但需要保证注册中心的健康检查探活是通畅的。

### Q: Unrecognized VM option 'CMSParallelRemarkEnabled' Error: Could not create the Java Virtual Machine. Error: A fatal exception has occurred. Program will exit.导致seata-server无法启动？
A: 这个是因为使用了高版本的jdk导致。高版本的jdk取消了cms处理器，转而采用了zgc代替他。 

解决方案有两个: 
1. 降级jdk版本 
2. 在seata的启动脚本中删除cms的jdk命令

### Q: Seata的SQL支持范围？
A: 请参考附录[SQL参考](http://seata.io/zh-cn/docs/user/sqlreference/sql-restrictions.html)

### Q: Seata的JDK版本要求？
A: 目前Seata支持的JDK版本为JDK8、11。其余版本不确保100%兼容。

### Q: Oracle的NUMBER长度超过19之后，实体使用Long映射，导致获取不到行信息，导致undo_log无法插入，也无法回滚？
A: Oracle的NUMBER长度超过19之后，用Long的话，setObject会查不出数据来，将实体的Long修改为BigInteger或BigDecimal即可解决问题。

### Q: 怎么处理 io.seata.rm.datasource.exec.LockConflictException: get global lock fail
A: 获取全局锁失败，一般是出现分布式资源竞争导致，请保证你竞争资源的周期是合理的，并且在业务上做好重试。
当一个全局事务因为获取锁失败的时候，应该重新完整地从`@Globaltransational`的TM端重新发起。

Seata提供了一个“全局锁重试”功能，默认未开启，可以通过下面这个配置来开启。
```properties
#遇到全局锁冲突时是否回滚，默认为true
client.rm.lock.retryPolicyBranchRollbackOnConflict=false
```
开启后，默认的全局锁重试逻辑是：线程sleep 10ms，再次争全局锁，最多30次
```properties
#你可通过这2个配置来修改锁重试机制
client.rm.lock.retryInterval=10
client.rm.lock.retryTimes=30
```
另外，你也可以直接在@GlobalTransactional上单独配置重试逻辑，优先级比Seata全局配置更高
```java
@GlobalTransactional(lockRetryInternal = 100, lockRetryTimes = 30)  // v1.4.2
@GlobalTransactional(lockRetryInterval = 100, lockRetryTimes = 30)  // v1.5
```

### Q：为什么在客户端在编译和运行时 JDK 版本都是 1.8 的情况下还会出现 java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer 错误 ?
A: 这是因为编译了 seata 源码然后覆盖了本地的 seata 依赖包的原因，在编译 seata 源码时使用了 JDK 11，而在 JDK 11 中由于改写了 `flip()` 方法，所以导致不兼容。

解决办法：
* 编译 seata 源码时确认 JDK 版本为 1.8，以免导致兼容问题
* 如果已经用 JDK 11 编译了 seata 的源码，请删除本地 maven 仓库下 io.seata 路径下所有包。然后重新编译你的项目，让项目重新拉取中央仓库的 seata 的依赖包

### Q：为什么在使用Apple的M1芯片下载maven依赖时，无法下载依赖`com.google.protobuf:protoc:exe:3.3.0` ？
A: 在`serializer/seata-serializer-protobuf/pom.xml`文件中，依赖版本是通过识别操作系统变量定义的：`com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}`。 
在远程仓库中，不存在Apple的M1芯片架构对应的依赖版本。

解决方案： 将上述依赖改写为固定版本：`com.google.protobuf:protoc:3.3.0:exe:osx-x86_64`，即可到远程仓库下载对应版本依赖。



## 文档

* [官网](https://seata.io/zh-cn/docs/overview/what-is-seata.html)
  * [Seata事务隔离](https://seata.io/zh-cn/docs/user/appendix/isolation.html) - `原理`/`防止脏读`/`防止脏写`/`源码分析`【重要】
  * [API 支持](https://seata.io/zh-cn/docs/user/api.html)
    * `GlobalTransaction` - 全局事务：包括开启事务、提交、回滚、获取当前状态等方法。
    * `GlobalTransactionContext` - GlobalTransaction 实例的获取需要通过 GlobalTransactionContext。
    * `TransactionalTemplate` - 事务化模板：通过上述 GlobalTransaction 和 GlobalTransactionContext API 把一个业务服务的调用包装成带有分布式事务支持的服务。
    * `RootContext` - 事务的根上下文：负责在应用的运行时，维护 XID；远程调用事务上下文的传播；事务的暂停和恢复。
  * [微服务框架支持](https://seata.io/zh-cn/docs/user/microservice.html)
    * 事务上下文
    * 事务传播
    * 对 Dubbo 支持的解读
  * SQL
    * [SQL限制](https://seata.io/zh-cn/docs/user/sqlreference/sql-restrictions.html)
    * [DML语句](https://seata.io/zh-cn/docs/user/sqlreference/dml.html)
    * [SQL修饰](https://seata.io/zh-cn/docs/user/sqlreference/sql-decoration.html)
    * [函数](https://seata.io/zh-cn/docs/user/sqlreference/function.html)
  * APM
    * [SkyWalking](https://seata.io/zh-cn/docs/user/apm/skywalking.html)
    * [Prometheus](https://seata.io/zh-cn/docs/user/apm/prometheus.html)
    * [Metrics](https://seata.io/zh-cn/docs/dev/seata-mertics.html)
  * [版本升级指南](https://seata.io/zh-cn/docs/ops/upgrade.html)
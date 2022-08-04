# ShardingSphere

Ecosystem to transform any database into a distributed database system, and enhance it with sharding, elastic scaling, encryption features & more

Apache ShardingSphere 认为数据库是一个`存储节点`，ShardingSphere-JDBC 和 ShardingSphere-Proxy 是数据库集群中的`计算节点`，用户在使用时，
将通过完整的集群视角看待分布式数据库。通过 Apache ShardingSphere 可以直接操作相关数据，而不需要跨过它直接访问存储节点。

| Solutions/Features  |  Distributed Database   |    Data Security     |         Database Gateway          | Stress Testing  |
|:-------------------:|:-----------------------:|:--------------------:|:---------------------------------:|:---------------:|
|        |      Data Sharding      |     Data Encrypt     | Heterogeneous Databases Supported | Shadow Database |
|        |   Readwrite-splitting   | Row Authority (TODO) |   SQL Dialect Translate (TODO)    |  Observability  |
|        | Distributed Transaction |   SQL Audit (TODO)   |                                   |                 |
|        |    Elastic Scale-out    | SQL Firewall (TODO)  |                                   |                 |
|        |    Highly Available     |                      |                                   |                 |


## 1. ShardingSphere-JDBC

### YAML配置 - [链接](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/yaml-config/)

* `!!` 表示实例化该类
* `!` 表示自定义别名

### JDBC 驱动 - [链接](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/jdbc-driver/)

### Spring Boot Starter - [链接](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/spring-boot-starter/)

> 注：`行表达式`标识符可以使用 `${...}` 或 `$->{...}`，但前者与 Spring 本身的属性文件占位符冲突，因此在 Spring 环境中使用行表达式标识符建议使用 `$->{...}`。

### 属性配置

Apache ShardingSphere 提供属性配置的方式配置系统级配置。

| 名称	|  数据类型   | 说明	| 默认值 |
|:-------------------:|:-------:|:-------------------:|:-------------------:|
|sql-show (?)|	boolean |	是否在日志中打印 SQL。帮助开发者快速定位系统问题。日志内容包含：逻辑 SQL，真实 SQL 和 SQL 解析结果。 如果开启配置，日志将使用 Topic ShardingSphere-SQL，日志级别是 INFO	|false|
|sql-simple (?)|	boolean	|是否在日志中打印简单风格的 SQL	|false|
|kernel-executor-size (?)|  	int	  |用于设置任务处理线程池的大小 每个 ShardingSphereDataSource 使用一个独立的线程池，同一个 JVM 的不同数据源不共享线程池	|infinite|
|max-connections-size-per-query (?)|  	int   |	一次查询请求在每个数据库实例中所能使用的最大连接数|	1|
|check-table-metadata-enabled (?)|	boolean	|在程序启动和更新时，是否检查分片元数据的结构一致性|	false|
|sql-federation-enabled (?)	|boolean	 |是否开启联邦查询	|false|

```yaml
props:
    sql-show: true
```

> 源码参考: `org.apache.shardingsphere.infra.config.props.ConfigurationPropertyKey`

### 内置算法

* [元数据持久化仓库](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/metadata-repository/)
  * H2 数据库持久化
  * ZooKeeper 持久化
  * Etcd 持久化
* [分片算法](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/sharding/)
  * 自动分片算法
    * `MOD` - 取模分片算法
    * `HASH_MOD` - 哈希取模分片算法
    * `VOLUME_RANGE` - 基于分片容量的范围分片算法
    * `BOUNDARY_RANGE` - 基于分片边界的范围分片算法
    * `AUTO_INTERVAL` - 自动时间段分片算法
  * 标准分片算法
    * `INLINE` - 行表达式分片算法
    * `INTERVAL` - 时间范围分片算法
  * 复合分片算法
    * `COMPLEX_INLINE` - 复合行表达式分片算法
  * Hint 分片算法
    * `HINT_INLINE` - Hint 行表达式分片算法
  * `CLASS_BASED` - 自定义类分片算法
* [分布式序列算法](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/keygen/)
  * `SNOWFLAKE` - 雪花算法
  * `UUID`
  * `COSID`
  * `CosId-Snowflake`
* [负载均衡算法](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/load-balance/)
  * `ROUND_ROBIN` - 事务内，读请求路由到 primary，事务外，采用轮询策略路由到 replica。（默认算法）
  * `RANDOM` - 	事务内，读请求路由到 primary，事务外，采用随机策略路由到 replica。
  * `WEIGHT` - 事务内，读请求路由到 primary，事务外，采用权重策略路由到 replica。需配置属性，属性名：${replica-name}，数据类型：double, 
  属性名字使用读库名字，参数填写读库对应的权重值。权重参数范围最小值 > 0，合计 <= Double.MAX_VALUE。
  * `TRANSACTION_RANDOM` - 显示/非显示开启事务，读请求采用随机策略路由到多个 replica。
  * `TRANSACTION_ROUND_ROBIN` - 显示/非显示开启事务，读请求采用轮询策略路由到多个 replica。
  * `TRANSACTION_WEIGHT` - 显示/非显示开启事务，读请求采用权重策略路由到多个 replica。需配置属性，属性名：${replica-name}，数据类型：double, 
  属性名字使用读库名字，参数填写读库对应的权重值。权重参数范围最小值 > 0，合计 <= Double.MAX_VALUE。
  * `FIXED_REPLICA_RANDOM` - 显示开启事务，读请求采用随机策略路由到一个固定 replica；不开事务，每次读流量使用随机策略路由到不同的 replica。
  * `FIXED_REPLICA_ROUND_ROBIN` - 显示开启事务，读请求采用轮询策略路由到一个固定 replica；不开事务，每次读流量使用轮询策略路由到不同的 replica。
  * `FIXED_REPLICA_WEIGHT` - 	显示开启事务，读请求采用权重策略路由到一个固定 replica；不开事务，每次读流量使用权重策略路由到不同的 replica。
    需配置属性，属性名：${replica-name}，数据类型：double, 属性名字使用读库名字，参数填写读库对应的权重值。权重参数范围最小值 > 0，合计 <= Double.MAX_VALUE。
  * `FIXED_PRIMARY` - 读请求全部路由到 primary。
* [加密算法](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/encrypt/)
  * `MD5`
  * `AES`
  * `RC4`
  * `SM3`
  * `SM4`
* [影子算法](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/shadow/)
  * 列影子算法
    * `VALUE_MATCH` - 列值匹配算法
    * `REGEX_MATCH` - 列正则表达式匹配算法
  * Hint 影子算法
    * `SIMPLE_HINT` - 简单 Hint 匹配影子算法
* [SQL 翻译](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/sql-translator/)
  * `NATIVE` - 原生 SQL 翻译器
  * `JOOQ` - 使用 JooQ 的 SQL 翻译器

> 注：ShardingSphere-JDBC - [不支持项](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/unsupported/)

## 2. ShardingSphere-Proxy

* [DistSQL](https://shardingsphere.apache.org/document/current/cn/features/distsql/)
  * RDL - `Resource & Rule Definition Language`
  * RQL - `Resource & Rule Query Language`
  * RAL - `Resource & Rule Administration Language`
* [DistSQL 详解](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-proxy/distsql/)
* [DistSQL 技术参考](https://shardingsphere.apache.org/document/current/cn/reference/distsql/)

### 弹性伸缩

ShardingSphere-Scaling 是一个提供给用户的通用的 ShardingSphere 数据接入迁移，及弹性伸缩的解决方案。

* [弹性伸缩](https://shardingsphere.apache.org/document/current/cn/features/scaling/)
* [运行部署](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-proxy/scaling/build/)
* [使用手册](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-proxy/scaling/usage/)
* [弹性伸缩 - 技术参考](https://shardingsphere.apache.org/document/current/cn/reference/scaling/)

## 3. ShardingSphere-Sidecar

> 规划中

## 4. 重点功能

### 数据分片
* [数据分片](https://shardingsphere.apache.org/document/current/cn/features/sharding/)
  * [表](https://shardingsphere.apache.org/document/current/cn/features/sharding/concept/table/)
    * 逻辑表
    * 真实表
    * 绑定表
    * 广播表
    * 单表
  * [分片](https://shardingsphere.apache.org/document/current/cn/features/sharding/concept/sharding/)
    * 分片键
    * 分片算法
      * 自动化分片算法
      * 自定义分片算法 - `标准分片算法`/`复合分片算法`/`Hint 分片算法`
    * 分片策略
    * 强制分片路由
  * [行表达式](https://shardingsphere.apache.org/document/current/cn/features/sharding/concept/inline-expression/)
  * [分布式主键](https://shardingsphere.apache.org/document/current/cn/features/sharding/concept/key-generator/)
    * UUID
    * NanoID
    * SNOWFLAKE
    * 时钟回拨
  * [SQL 支持程度](https://shardingsphere.apache.org/document/current/cn/features/sharding/use-norms/sql/)
  * [分页](https://shardingsphere.apache.org/document/current/cn/features/sharding/use-norms/pagination/)
    * 性能瓶颈
    * ShardingSphere 的优化
    * 分页方案优化

* [数据分片 - 【原理】](https://shardingsphere.apache.org/document/current/cn/reference/sharding/)
  * 解析引擎
  * 路由引擎
    * 分片路由 - `直接路由`/`标准路由`/`笛卡尔路由`
    * 广播路由 - `全库表路由`/`全库路由`/`全实例路由`/`单播路由`/`阻断路由`
  * 改写引擎
    * 正确性改写 - `标识符改写`/`补列`/`分页修正`/`批量拆分`
    * 优化改写 - `单节点优化`/`流式归并优化`
  * 执行引擎
    * 连接模式 - `内存限制模式`/`连接限制模式`
    * 自动化执行引擎 - `准备阶段`/`执行阶段`
  * 归并引擎
    * 遍历归并
    * 排序归并
    * 分组归并
    * 聚合归并
    * 分页归并

### 读写分离/高可用

* [读写分离](https://shardingsphere.apache.org/document/current/cn/features/readwrite-splitting/)
* [高可用](https://shardingsphere.apache.org/document/current/cn/features/ha/)

### 强制路由

* [强制分片路由](https://shardingsphere.apache.org/document/current/cn/features/sharding/concept/hint/)
* [数据分片 - 强制路由实现](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/sharding/hint/)
* [读写分离 - 强制路由](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/readwrite-splitting/hint/)

### 分布式事务

* [分布式事务](https://shardingsphere.apache.org/document/current/cn/features/transaction/)
  * LOCAL 事务
  * XA 事务
  * BASE 事务
* [分布式事务集成](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/)
  * [Java API](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/java-api/)
  * [Spring Boot Starter](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/spring-boot-starter/)
  * [Spring Namespace](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/spring-namespace/)
  * [XA - Atomikos](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/atomikos/)
  * [XA - Narayana](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/narayana/)
  * [XA - Bitronix](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/bitronix/)
  * [BASE - Seata](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/transaction/seata/)
* 分布式事务 -【原理】
  * [XA 事务](https://shardingsphere.apache.org/document/current/cn/reference/transaction/2pc-xa-transaction/)
  * [SEATA 柔性事务](https://shardingsphere.apache.org/document/current/cn/reference/transaction/base-transaction-seata/)


### 可观察性

* [可观察性](https://shardingsphere.apache.org/document/current/cn/features/observability/)
  * Tracing
  * Metrics
  * Logging
* [使用探针](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/observability/agent/)
* [应用性能监控集成](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/special-api/observability/apm-integration/)

### 影子库

* [影子库](https://shardingsphere.apache.org/document/current/cn/features/shadow/)
* [影子库 - 技术参考](https://shardingsphere.apache.org/document/current/cn/reference/shadow/)

### 数据加密

* [数据加密](https://shardingsphere.apache.org/document/current/cn/features/encrypt/)
* [数据加密 - 技术参考](https://shardingsphere.apache.org/document/current/cn/reference/encrypt/)
* [聊聊 Sharding-JDBC 数据脱敏](https://netsecurity.51cto.com/article/709033.html) - 辅助查询列

## 5. 开发者手册

Apache ShardingSphere 可插拔架构提供了数十个基于 SPI 的扩展点。对于开发者来说，可以十分方便的对功能进行定制化扩展。

> [官方文档](https://shardingsphere.apache.org/document/current/cn/dev-manual/)

### 运行模式 SPI

* `StandalonePersistRepository` - Standalone 模式配置信息持久化
  * `FileRepository` - 基于 File 的持久化
* `ClusterPersistRepository` - Cluster 模式配置信息持久化
  * `CuratorZookeeperRepository` - 基于 ZooKeeper 的持久化
  * `EtcdRepository` - 基于 Etcd 的持久化
* `GovernanceWatcher` - 治理监听器
  * `ComputeNodeStateChangedWatcher` - 计算节点状态变化监听器
  * `DatabaseLockChangedWatcher` - 数据库锁状态变化监听器
  * `DistributedLockChangedWatcher` - 分布式锁变化监听器
  * `GlobalRuleChangedWatcher` - 全局规则配置变化监听器
  * `MetaDataChangedWatcher` - 元数据变化监听器
  * `PropertiesChangedWatcher` - 属性变化监听器
  * `StorageNodeStateChangedWatcher` - 存储节点状态变化监听器

### 配置 SPI

* `RuleBuilder` - 用于将用户配置转化为规则对象的接口
  * `AuthorityRuleConfiguration` - 用于将权限用户配置转化为权限规则对象	AuthorityRuleBuilder
  * `SQLParserRuleConfiguration` - 用于将 SQL 解析用户配置转化为 SQL 解析规则对象	SQLParserRuleBuilder
  * `TransactionRuleConfiguration` - 用于将事务用户配置转化为事务规则对象	TransactionRuleBuilder
  * `SingleTableRuleConfiguration` - 用于将单表用户配置转化为单表规则对象	SingleTableRuleBuilder
  * `ShardingRuleConfiguration` - 用于将分片用户配置转化为分片规则对象	ShardingRuleBuilder
  * `AlgorithmProvidedShardingRuleConfiguration` - 用于将基于算法的分片用户配置转化为分片规则对象	AlgorithmProvidedShardingRuleBuilder
  * `ReadwriteSplittingRuleConfiguration` - 用于将读写分离用户配置转化为读写分离规则对象	ReadwriteSplittingRuleBuilder
  * `AlgorithmReadwriteSplittingRuleConfiguration` - 用于将基于算法的读写分离用户配置转化为读写分离规则对象	AlgorithmProvidedReadwriteSplittingRuleBuilder
  * `DatabaseDiscoveryRuleConfiguration` - 用于将数据库发现用户配置转化为数据库发现规则对象	DatabaseDiscoveryRuleBuilder
  * `AlgorithmDatabaseDiscoveryRuleConfiguration` - 用于将基于算法的数据库发现用户配置转化为数据库发现规则对象	AlgorithmProvidedDatabaseDiscoveryRuleBuilder
  * `EncryptRuleConfiguration` - 用于将加密用户配置转化为加密规则对象	EncryptRuleBuilder
  * `AlgorithmEncryptRuleConfiguration` - 用于将基于算法的加密用户配置转化为加密规则对象	AlgorithmProvidedEncryptRuleBuilder
  * `ShadowRuleConfiguration` - 用于将影子库用户配置转化为影子库规则对象	ShadowRuleBuilder
  * `AlgorithmShadowRuleConfiguration` - 用于将基于算法的影子库用户配置转化为影子库规则对象	AlgorithmProvidedShadowRuleBuilder
* `YamlRuleConfigurationSwapper` - 用于将 YAML 配置转化为标准用户配置
  * `AUTHORITY` - 用于将权限规则的 YAML 配置转化为权限规则标准配置	AuthorityRuleConfigurationYamlSwapper
  * `SQL_PARSER` - 用于将 SQL 解析的 YAML 配置转化为 SQL 解析标准配置 SQLParserRuleConfigurationYamlSwapper
  * `TRANSACTION` - 用于将事务的 YAML 配置转化为事务标准配置	TransactionRuleConfigurationYamlSwapper
  * `SINGLE` - 用于将单表的 YAML 配置转化为单表标准配置	SingleTableRuleConfigurationYamlSwapper
  * `SHARDING` - 用于将分片的 YAML 配置转化为分片标准配置	ShardingRuleConfigurationYamlSwapper
  * `SHARDING` - 用于将基于算法的分片配置转化为分片标准配置 ShardingRuleAlgorithmProviderConfigurationYamlSwapper
  * `READWRITE_SPLITTING` - 用于将读写分离的 YAML 配置转化为读写分离标准配置 ReadwriteSplittingRuleConfigurationYamlSwapper
  * `READWRITE_SPLITTING` - 用于将基于算法的读写分离配置转化为读写分离标准配置	ReadwriteSplittingRuleAlgorithmProviderConfigurationYamlSwapper
  * `DB_DISCOVERY` - 用于将数据库发现的 YAML 配置转化为数据库发现标准配置 DatabaseDiscoveryRuleConfigurationYamlSwapper
  * `DB_DISCOVERY` - 用于将基于算法的数据库发现配置转化为数据库发现标准配置 DatabaseDiscoveryRuleAlgorithmProviderConfigurationYamlSwapper
  * `ENCRYPT` - 用于将加密的 YAML 配置转化为加密标准配置 EncryptRuleConfigurationYamlSwapper
  * `ENCRYPT` - 用于将基于算法的加密配置转化为加密标准配置 EncryptRuleAlgorithmProviderConfigurationYamlSwapper
  * `SHADOW` - 用于将影子库的 YAML 配置转化为影子库标准配置 ShadowRuleConfigurationYamlSwapper
  * `SHADOW` - 用于将基于算法的影子库配置转化为影子库标准配置	ShadowRuleAlgorithmProviderConfigurationYamlSwapper
  * `SQL_TRANSLATOR` - 用于将 SQL 转换的 YAML 配置转化为 SQL 转换标准配置 SQLTranslatorRuleConfigurationYamlSwapper
* `ShardingSphereYamlConstruct` - 用于将定制化对象和 YAML 相互转化
  * `YamlNoneShardingStrategyConfiguration` - 用于将不分片策略对象和 YAML 相互转化 NoneShardingStrategyConfigurationYamlConstruct

### 内核 SPI

* `SQLRouter` - 用于处理路由结果
  * `ReadwriteSplittingSQLRouter` - 用于处理读写分离路由结果
  * `DatabaseDiscoverySQLRouter` - 用于处理数据库发现路由结果
  * `SingleTableSQLRouter` - 用于处理单表路由结果
  * `ShardingSQLRouter` - 用于处理分片路由结果
  * `ShadowSQLRouter` - 用于处理影子库路由结果
* `SQLRewriteContextDecorator` - 用于处理 SQL 改写结果
  * `ShardingSQLRewriteContextDecorator` - 用于处理分片 SQL 改写结果
  * `EncryptSQLRewriteContextDecorator` - 用于处理加密 SQL 改写结果
* `SQLExecutionHook` - SQL 执行过程监听器
  * `TransactionalSQLExecutionHook` - 基于事务的 SQL 执行过程监听器
* `ResultProcessEngine` - 用于处理结果集
  * `ShardingResultMergerEngine` - 用于处理分片结果集归并
  * `EncryptResultDecoratorEngine` - 用于处理加密结果集改写
* `StoragePrivilegeHandler` - 使用数据库方言处理权限信息
  * `PostgreSQLPrivilegeHandler` - 使用 PostgreSQL 方言处理权限信息
  * `SQLServerPrivilegeHandler` - 使用 SQLServer 方言处理权限信息
  * `OraclePrivilegeHandler` - 使用 Oracle 方言处理权限信息
  * `MySQLPrivilegeHandler` - 使用 MySQL 方言处理权限信息
* `DynamicDataSourceStrategy` - 动态数据源获取策略
  * `DatabaseDiscoveryDynamicDataSourceStrategy` - 使用数据库自动发现的功能获取动态数据源

### 数据源 SPI

* `DatabaseType` - 支持的数据库类型
  * `SQL92DatabaseType` - 遵循 SQL92 标准的数据库类型
  * `MySQLDatabaseType` - MySQL 数据库
  * `MariaDBDatabaseType` - MariaDB 数据库
  * `PostgreSQLDatabaseType` - PostgreSQL 数据库
  * `OracleDatabaseType` - Oracle 数据库
  * `SQLServerDatabaseType` - SQLServer 数据库
  * `H2DatabaseType` - H2 数据库
  * `OpenGaussDatabaseType` - OpenGauss 数据库
* `DialectTableMetaDataLoader` - 用于使用数据库方言快速加载元数据
  * `MySQLTableMetaDataLoader` - 使用 MySQL 方言加载元数据
  * `OracleTableMetaDataLoader` - 使用 Oracle 方言加载元数据
  * `PostgreSQLTableMetaDataLoader` - 使用 PostgreSQL 方言加载元数据
  * `SQLServerTableMetaDataLoader` - 使用 SQLServer 方言加载元数据
  * `H2TableMetaDataLoader` - 使用 H2 方言加载元数据
  * `OpenGaussTableMetaDataLoader` - 使用 OpenGauss 方言加载元数据
* `DataSourcePoolMetaData` - 数据源连接池元数据
  * `DBCPDataSourcePoolMetaData` - DBCP 数据库连接池元数据
  * `HikariDataSourcePoolMetaData` - Hikari 数据源连接池元数据
* `DataSourcePoolActiveDetector` - 数据源连接池活跃探测器
  * `DefaultDataSourcePoolActiveDetector` - 默认数据源连接池活跃探测器
  * `HikariDataSourcePoolActiveDetector` - Hikari 数据源连接池活跃探测器

### SQL 解析 SPI

* `DatabaseTypedSQLParserFacade` - 配置用于 SQL 解析的词法分析器和语法分析器入口
  * `MySQLParserFacade` - 基于 MySQL 的 SQL 解析器入口
  * `PostgreSQLParserFacade` - 基于 PostgreSQL 的 SQL 解析器入口
  * `SQLServerParserFacade` - 基于 SQLServer 的 SQL 解析器入口
  * `OracleParserFacade` - 基于 Oracle 的 SQL 解析器入口
  * `SQL92ParserFacade` - 基于 SQL92 的 SQL 解析器入口
  * `OpenGaussParserFacade` - 基于 openGauss 的 SQL 解析器入口
* `SQLVisitorFacade` - SQL 语法树访问器入口
  * `SQLVisitorFacade` - SQL 语法树访问器入口

### 代理端 SPI

* `DatabaseProtocolFrontendEngine` - 用于 ShardingSphere-Proxy 解析与适配访问数据库的协议
  * `MySQL` - MySQLFrontendEngine
  * `PostgreSQL` - PostgreSQLFrontendEngine
  * `openGauss` - OpenGaussFrontendEngine
* `AuthorityProvideAlgorithm` - 用户权限加载逻辑
  * `LL_PERMITTED` - 默认授予所有权限（不鉴权）	AllPermittedPrivilegesProviderAlgorithm
  * `DATABASE_PERMITTED` - 通过属性 user-database-mappings 配置的权限	DatabasePermittedPrivilegesProviderAlgorithm

### 数据分片 SPI

* `ShardingAlgorithm` - 分片算法
  * `BoundaryBasedRangeShardingAlgorithm` - 基于分片边界的范围分片算法
  * `VolumeBasedRangeShardingAlgorithm` - 基于分片容量的范围分片算法
  * `ComplexInlineShardingAlgorithm` - 基于行表达式的复合分片算法
  * `AutoIntervalShardingAlgorithm` - 基于可变时间范围的分片算法
  * `ClassBasedShardingAlgorithm` - 基于自定义类的分片算法
  * `HintInlineShardingAlgorithm` - 基于行表达式的 Hint 分片算法
  * `IntervalShardingAlgorithm` - 基于固定时间范围的分片算法
  * `HashModShardingAlgorithm` - 基于哈希取模的分片算法
  * `InlineShardingAlgorithm` - 基于行表达式的分片算法
  * `ModShardingAlgorithm` - 基于取模的分片算法
  * `CosIdModShardingAlgorithm` - 基于 CosId 的取模分片算法
  * `CosIdIntervalShardingAlgorithm` - 基于 CosId 的固定时间范围的分片算法
  * `CosIdSnowflakeIntervalShardingAlgorithm` - 基于 CosId 的雪花ID固定时间范围的分片算法
* `KeyGenerateAlgorithm` - 分布式主键生成算法
  * `SnowflakeKeyGenerateAlgorithm` - 基于雪花算法的分布式主键生成算法
  * `UUIDKeyGenerateAlgorithm` - 基于 UUID 的分布式主键生成算法
  * `CosIdKeyGenerateAlgorithm` - 基于 CosId 的分布式主键生成算法
  * `CosIdSnowflakeKeyGenerateAlgorithm` - 基于 CosId 的雪花算法分布式主键生成算法
  * `NanoIdKeyGenerateAlgorithm` - 基于 NanoId 的分布式主键生成算法
* `ShardingAuditAlgorithm` - 分片审计算法
  * `DMLShardingConditionsShardingAuditAlgorithm` - 禁止不带分片键的DML审计算法
* `DatetimeService` - 获取当前时间进行路由
  * `DatabaseDatetimeServiceDelegate` - 从数据库中获取当前时间进行路由
  * `SystemDatetimeService` - 从应用系统时间中获取当前时间进行路由
* `DatabaseSQLEntry` - 获取当前时间的数据库方言
  * `MySQLDatabaseSQLEntry` - 从 MySQL 获取当前时间的数据库方言
  * `PostgreSQLDatabaseSQLEntry` - 从 PostgreSQL 获取当前时间的数据库方言
  * `OracleDatabaseSQLEntry` - 从 Oracle 获取当前时间的数据库方言
  * `SQLServerDatabaseSQLEntry` - 从 SQLServer 获取当前时间的数据库方言

### 读写分离 SPI

* `ReadQueryLoadBalanceAlgorithm` - 读库负载均衡算法
  * `RoundRobinReplicaLoadBalanceAlgorithm` - 基于轮询的读库负载均衡算法（默认算法）
  * `RandomReplicaLoadBalanceAlgorithm` - 基于随机的读库负载均衡算法
  * `WeightReplicaLoadBalanceAlgorithm` - 基于权重的读库负载均衡算法
  * `TransactionRandomReplicaLoadBalanceAlgorithm` - 无论是否在事务中，读请求采用随机策略路由到多个读库
  * `TransactionRoundRobinReplicaLoadBalanceAlgorithm` - 无论是否在事务中，读请求采用轮询策略路由到多个读库
  * `TransactionWeightReplicaLoadBalanceAlgorithm` - 无论是否在事务中，读请求采用权重策略路由到多个读库
  * `FixedReplicaRandomLoadBalanceAlgorithm` - 显示开启事务，读请求采用随机策略路由到一个固定读库；不开事务，每次读流量使用指定算法路由到不同的读库
  * `FixedReplicaRoundRobinLoadBalanceAlgorithm` - 显示开启事务，读请求采用轮询策略路由到一个固定读库；不开事务，每次读流量使用指定算法路由到不同的读库
  * `FixedReplicaWeightLoadBalanceAlgorithm` - 显示开启事务，读请求采用权重策略路由到多个读库；不开事务，每次读流量使用指定算法路由到不同的读库
  * `FixedPrimaryLoadBalanceAlgorithm` - 读请求全部路由到主库

### 高可用 SPI

* `DatabaseDiscoveryProviderAlgorithm` - 数据库发现算法
  * `MGRDatabaseDiscoveryProviderAlgorithm` - 基于 MySQL MGR 的数据库发现算法
  * `MySQLNormalReplicationDatabaseDiscoveryProviderAlgorithm` - 基于 MySQL 主从同步的数据库发现算法
  * `OpenGaussNormalReplicationDatabaseDiscoveryProviderAlgorithm` - 基于 openGauss 主从同步的数据库发现算法

### 分布式事务 SPI

* `ShardingSphereTransactionManager` - 分布式事务管理器
  * `XA` - 基于 XA 的分布式事务管理器
  * `BASE` - 基于 Seata 的分布式事务管理器
* `XATransactionManagerProvider` - XA 分布式事务管理器
  * `Atomikos` - 基于 Atomikos 的 XA 分布式事务管理器
  * `Narayana` - 基于 Narayana 的 XA 分布式事务管理器
  * `Bitronix` - 基于 Bitronix 的 XA 分布式事务管理器
* `XADataSourceDefinition` - 用于非 XA 数据源转化为 XA 数据源
  * MySQL
  * MariaDB
  * Oracle
  * SQLServer
  * H2
* `DataSourcePropertyProvider` - 用于获取数据源连接池的标准属性
  * `com.zaxxer.hikari.HikariDataSource` - 用于获取 HikariCP 连接池的标准属性

### SQL 检查 SPI

* `SQLChecker` - SQL 检查器
  * `AuthorityChecker` - 权限检查器
  * `ShardingAuditChecker` - 分片审计检查器

### 数据加密 SPI

* `EncryptAlgorithm` - 数据加密算法
  * `MD5EncryptAlgorithm` - 基于 MD5 的数据加密算法
  * `AESEncryptAlgorithm` - 	基于 AES 的数据加密算法
  * `RC4EncryptAlgorithm` - 基于 RC4 的数据加密算法
  * `SM4EncryptAlgorithm` - 基于 SM4 的数据加密算法
  * `SM3EncryptAlgorithm` - 基于 SM3 的数据加密算法

### 影子库 SPI

* `ShadowAlgorithm` - 影子库路由算法
  * `ColumnValueMatchShadowAlgorithm` - 基于字段值匹配影子算法
  * `ColumnRegexMatchShadowAlgorithm` - 基于字段值正则匹配影子算法
  * `SimpleHintShadowAlgorithm` - 基于 Hint 简单匹配影子算法

### 可观察性 SPI

* `PluginBootService` - 插件启动服务定义接口
  * Prometheus
  * Logging
  * Jaeger
  * OpenTelemetry
  * OpenTracing
  * Zipkin
* `PluginDefinitionService` - 探针插件定义服务接口
  * Prometheus
  * Logging
  * Jaeger
  * OpenTelemetry
  * OpenTracing
  * Zipkin

## 6. 注意事项

> `DataSourceMetaData.getDefaultQueryProperties()` - 提供了各数据库类型的**默认属性**，提供了非常重要的参考意义。

> `DataSourcePoolCreator.create()` - 数据源创建细节

## 7. 参考文档

* [注册中心数据结构 -【重要】](https://shardingsphere.apache.org/document/current/cn/reference/management/)
* [FAQ](https://shardingsphere.apache.org/document/current/cn/reference/faq/)
* [官方示例](https://github.com/apache/shardingsphere/tree/master/examples)
* [A Holistic Pluggable Platform for Data Sharding — ICDE 2022 & Understanding](https://faun.pub/a-holistic-pluggable-platform-for-data-sharding-icde-2022-understanding-apache-shardingsphere-55779cfde16)
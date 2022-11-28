# Kafka安装及配置

## 安装

### 1. 安装`Java 8+`及系统配置

先决条件：
* [Java Version](https://kafka.apache.org/documentation/#java)
* [Hardware and OS](https://kafka.apache.org/documentation/#hwandos)

```bash
# 安装 Java 17
shell> wget https://download.oracle.com/java/17/archive/jdk-17.0.5_linux-x64_bin.tar.gz
shell> tar -zxf jdk-17.0.5_linux-x64_bin.tar.gz
shell> # 修改 JAVA_HOME 及 PATH
shell> # 修改 fs.file-max 及 ulimit nofile
```

系统配置：
* fs.file-max
  * `sysctl -w fs.file-max=655350`(Session)
  * `fs.file-max = 655350` - `/etc/sysctl.conf`(永久)
* nofile
  * `ulimit -n 65536`(Session)
  * `* - nofile 65536` - `/etc/security/limits.conf`(永久)
* vm.max_map_count
  * `sysctl -w vm.max_map_count=655350`(Session)
  * `vm.max_map_count = 655350` - `/etc/sysctl.conf`(永久)
* 文件系统
  * 常用的文件系统有：`EXT4`、`XFS`，推荐使用`XFS`
  * `noatime` - This option disables updating of a file's atime (last access time) attribute when the file is read. 
  This can eliminate a significant number of filesystem writes, especially in the case of bootstrapping consumers. Kafka does not rely on the atime attributes at all, so it is safe to disable this.

> 注：系统配置值实际情况而定，详情参考上述官网文档链接。

### 2. 下载 Kafka

```bash
shell> wget https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
shell> tar -xzf kafka_2.13-3.3.1.tgz
shell> cd kafka_2.13-3.3.1
```

### 3. 配置

#### 3.1 [Broker Configs](https://kafka.apache.org/documentation/#brokerconfigs)

From `Kafka version 1.1` onwards, some of the broker configs can be updated without restarting the broker. 
See the `Dynamic Update Mode` column in [Broker Configs](https://kafka.apache.org/documentation/#brokerconfigs) for the update mode of each broker config.
* `read-only`: Requires a broker restart for update
* `per-broker`: May be updated dynamically for each broker
* `cluster-wide`: May be updated dynamically as a cluster-wide default. May also be updated as a per-broker value for testing.

All configs that are configurable at cluster level may also be configured at per-broker level (e.g. for testing). 
If a config value is defined at different levels, the following **order** of precedence is used:
* Dynamic per-broker configs
* Dynamic cluster-wide default configs
* Static broker configs from server.properties
* Kafka default, see [broker configs](https://kafka.apache.org/documentation/#brokerconfigs)

Dynamic configs are stored in Kafka as `cluster metadata`. In ZooKeeper mode, dynamic configs are stored in **ZooKeeper**. 
In KRaft mode, dynamic configs are stored as records in the **metadata log**.

> 更多配置参考：https://github.com/apache/kafka/blob/trunk/core/src/main/scala/kafka/server/KafkaConfig.scala

> 注：**动态修改** Broker 配置[文档链接](https://kafka.apache.org/documentation/#dynamicbrokerconfigs)

#### 3.2 [Topic-Level Configs](https://kafka.apache.org/documentation/#topicconfigs)

> 注：Topic 配置也可以动态修改

#### 3.3 [Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)

#### 3.4 [Consumer Configs](https://kafka.apache.org/documentation/#consumerconfigs)

#### 3.5 [Kafka Connect Configs](https://kafka.apache.org/documentation/#connectconfigs)

* [Source Connector Configs](https://kafka.apache.org/documentation/#sourceconnectconfigs)
* [Sink Connector Configs](https://kafka.apache.org/documentation/#sinkconnectconfigs)

#### 3.6 [Kafka Streams Configs](https://kafka.apache.org/documentation/#streamsconfigs)

#### 3.7 [Admin Configs](https://kafka.apache.org/documentation/#adminclientconfigs)

#### 3.8 KRaft 模式配置

**Process Roles**

In KRaft mode each Kafka server can be configured as a controller, a broker, or both using the process.roles property. 
This property can have the following values:

* If `process.roles` is set to `broker`, the server acts as a broker.
* If `process.roles` is set to `controller`, the server acts as a controller.
* If `process.roles` is set to `broker,controller`, the server acts as both a broker and a controller.
* If `process.roles` is not set at all, it is assumed to be in `ZooKeeper mode`.

Kafka servers that act as both brokers and controllers are referred to as "combined" servers. Combined servers are simpler 
to operate for small use cases like a development environment. The key disadvantage is that the controller will be less isolated 
from the rest of the system. For example, it is not possible to roll or scale the controllers separately from the brokers 
in combined mode. Combined mode is not recommended in critical deployment environments.

### 4. 运行Kafka

#### 依托Zookeeper运行
```bash
# Start the ZooKeeper service
shell> bin/zookeeper-server-start.sh config/zookeeper.properties
# Start the Kafka broker service
shell> bin/kafka-server-start.sh config/server.properties
```

## 命令

### 查看指令帮助信息
```bash
shell> bin/xxx.sh --help
```

### 创建 Topic
```bash
shell> bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

### 生产事件
```bash
shell> bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
```
> 在控制台中输入的每行数据都会产生一个事件，按`Ctrl-C`停止producer运行

### 读取事件
```bash
shell> bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```
> 按`Ctrl-C`停止consumer运行

### 删除数据
```bash
shell> rm -rf /tmp/kafka-logs /tmp/zookeeper /tmp/kraft-combined-logs
```

## [APIS](https://kafka.apache.org/documentation/#api)

### 1. Producer API
The Producer API allows applications to send streams of data to topics in the Kafka cluster.

### 2. Consumer API
The Consumer API allows applications to read streams of data from topics in the Kafka cluster.

### 3. Streams API
The Streams API allows transforming streams of data from input topics to output topics.

### 4. Connect API
The Connect API allows implementing connectors that continually pull from some source system or application into Kafka or push from Kafka into some sink system or application.

### 5. Admin API
The Admin API allows managing and inspecting topics, brokers, and other Kafka objects.

## 参考文档

* [官方文档](https://kafka.apache.org/documentation/)
  * 4.DESIGN
    * [4.1 Motivation](https://kafka.apache.org/documentation/#majordesignelements)
    * [4.2 Persistence](https://kafka.apache.org/documentation/#persistence)
      * Don't fear the filesystem!
      * Constant Time Suffices
    * [4.3 Efficiency](https://kafka.apache.org/documentation/#maximizingefficiency)
      * End-to-end Batch Compression
    * [4.4 The Producer](https://kafka.apache.org/documentation/#theproducer)
      * Load balancing
      * Asynchronous send
    * [4.5 The Consumer](https://kafka.apache.org/documentation/#theconsumer)
      * Push vs. pull
      * Consumer Position
      * Offline Data Load
      * Static Membership
    * [4.6 Message Delivery Semantics](https://kafka.apache.org/documentation/#semantics)
    * [4.7 Replication](https://kafka.apache.org/documentation/#replication)
      * Replicated Logs: Quorums, ISRs, and State Machines (Oh my!)
      * Unclean leader election: What if they all die?
      * Availability and Durability Guarantees
      * Replica Management
    * [4.8 Log Compaction](https://kafka.apache.org/documentation/#compaction)
      * Log Compaction Basics
      * What guarantees does log compaction provide?
      * Log Compaction Details
      * Configuring The Log Cleaner
    * [4.9 Quotas](https://kafka.apache.org/documentation/#design_quotas)
      * Why are quotas necessary?
      * Client groups
      * Quota Configuration
      * Network Bandwidth Quotas
      * Request Rate Quotas
      * Enforcement
  * 5.IMPLEMENTATION
    * [5.1 Network Layer](https://kafka.apache.org/documentation/#networklayer)
    * [5.2 Messages](https://kafka.apache.org/documentation/#messages)
    * [5.3 Message Format](https://kafka.apache.org/documentation/#messageformat)
      * Record Batch / Control Batches
      * Record / Record Header
      * Old Message Format
    * [5.4 Log](https://kafka.apache.org/documentation/#log)
      * Writes
      * Reads
      * Deletes
      * Guarantees
    * [5.5 Distribution](https://kafka.apache.org/documentation/#distributionimpl)
      * Consumer Offset Tracking
      * ZooKeeper Directories
      * Cluster Id
  * 6.OPERATIONS
    * [6.1 Basic Kafka Operations](https://kafka.apache.org/documentation/#basic_ops)
      * Adding and removing topics
      * Modifying topics
      * **Graceful shutdown**
      * **Balancing leadership**
      * Balancing Replicas Across Racks
      * Mirroring data between clusters & Geo-replication
      * **Checking consumer position**
      * **Managing Consumer Groups**
      * [Add Reset Consumer Group Offsets tooling](https://cwiki.apache.org/confluence/display/KAFKA/KIP-122%3A+Add+Reset+Consumer+Group+Offsets+tooling)
      * Expanding your cluster
        * Automatically migrating data to new machines
        * Custom partition assignment and migration
      * Decommissioning brokers
      * Increasing replication factor
      * Limiting Bandwidth Usage during Data Migration
      * Setting quotas
    * [6.2 Datacenters](https://kafka.apache.org/documentation/#datacenters)
    * [6.3 Geo-Replication (Cross-Cluster Data Mirroring)](https://kafka.apache.org/documentation/#georeplication)
    * [6.4 Multi-Tenancy](https://kafka.apache.org/documentation/#multitenancy)
    * [6.5 Important Configs](https://kafka.apache.org/documentation/#config)
    * [Replace KRaft Controller Disk](https://kafka.apache.org/documentation/#replace_disk)
    * [6.8 Monitoring](https://kafka.apache.org/documentation/#monitoring)
    * [6.9 ZooKeeper Mode](https://kafka.apache.org/documentation/#zk)
    * [6.10 KRaft Mode](https://kafka.apache.org/documentation/#kraft) - 【重要】
      * Process Roles - 【重要】
      * Controllers - 【重要】
      * Storage Tool - 【重要】
      * Debugging
        * Metadata Quorum Tool
        * Dump Log Tool
        * Metadata Shell
      * Deploying Considerations
      * Missing Features
  * [7.SECURITY](https://kafka.apache.org/documentation/#security) - 【重要】
    * [7.3 Encryption and Authentication using SSL](https://kafka.apache.org/documentation/#security_ssl) - 使用`keytool`
      * 1.Generate SSL key and certificate for each Kafka broker
      * 2.Creating your own CA
      * 3.Signing the certificate
      * 4.Common Pitfalls in Production
      * 5.Configuring Kafka Brokers
      * 6.Configuring Kafka Clients
    * [7.4 Authentication using SASL](https://kafka.apache.org/documentation/#security_sasl)
    * [7.5 Authorization and ACLs](https://kafka.apache.org/documentation/#security_authz)
  * 8.KAFKA CONNECT
  * [9.KAFKA STREAMS](https://kafka.apache.org/documentation/streams)
    * [Play with a Streams Application](https://kafka.apache.org/33/documentation/streams/quickstart)
    * [Write your own Streams Applications](https://kafka.apache.org/33/documentation/streams/tutorial)
    * [Core Concepts](https://kafka.apache.org/33/documentation/streams/core-concepts)
    * [Architecture](https://kafka.apache.org/33/documentation/streams/architecture)
    * [Developer Manual](https://kafka.apache.org/33/documentation/streams/developer-guide)
      * [Writing a Streams Application](https://kafka.apache.org/33/documentation/streams/developer-guide/write-streams.html)
      * [Configuring a Streams Application](https://kafka.apache.org/33/documentation/streams/developer-guide/config-streams.html)
      * [Streams DSL](https://kafka.apache.org/33/documentation/streams/developer-guide/dsl-api.html)
      * [Processor API](https://kafka.apache.org/33/documentation/streams/developer-guide/processor-api.html)
      * [Naming Operators in a Streams DSL application](https://kafka.apache.org/33/documentation/streams/developer-guide/dsl-topology-naming.html)
      * [Data Types and Serialization](https://kafka.apache.org/33/documentation/streams/developer-guide/datatypes.html)
      * [Testing a Streams Application](https://kafka.apache.org/33/documentation/streams/developer-guide/testing.html)
      * [Interactive Queries](https://kafka.apache.org/33/documentation/streams/developer-guide/interactive-queries.html)
      * [Memory Management](https://kafka.apache.org/33/documentation/streams/developer-guide/memory-mgmt.html)
      * [Running Streams Applications](https://kafka.apache.org/33/documentation/streams/developer-guide/running-app.html)
      * [Managing Streams Application Topics](https://kafka.apache.org/33/documentation/streams/developer-guide/manage-topics.html)
      * [Streams Security](https://kafka.apache.org/33/documentation/streams/developer-guide/security.html)
      * [Application Reset Tool](https://kafka.apache.org/33/documentation/streams/developer-guide/app-reset-tool.html)
    * [Upgrade Guide](https://kafka.apache.org/33/documentation/streams/upgrade-guide)
* [kafka protocol guide](https://kafka.apache.org/protocol.html)
* [Kafka Ecosystem](https://cwiki.apache.org/confluence/display/KAFKA/Ecosystem)
* [Kafka Clients](https://cwiki.apache.org/confluence/display/KAFKA/Clients) - 开源社区
* [Compatibility Matrix](https://cwiki.apache.org/confluence/display/KAFKA/Compatibility+Matrix)
* [Kafka Improvement Proposals](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals) - 【重要】- 列举了所有改进提议，所有细节原理可从中获取
  * [KIP-500: Replace ZooKeeper with a Self-Managed Metadata Quorum](https://cwiki.apache.org/confluence/display/KAFKA/KIP-500%3A+Replace+ZooKeeper+with+a+Self-Managed+Metadata+Quorum)





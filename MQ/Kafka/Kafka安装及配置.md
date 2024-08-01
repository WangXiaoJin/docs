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

> 注：系统配置值**实际情况**而定，详情参考上述官网文档链接。

### 2. 下载 Kafka

```bash
shell> wget https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
shell> tar -xzf kafka_2.13-3.3.1.tgz
shell> cd kafka_2.13-3.3.1
```

### 3. 配置

通过以下三台机器集群部署：
* 192.168.202.245
* 192.168.202.246
* 192.168.202.247

#### 安全配置（通用配置）

**新增配置文件`kafka_server_jaas.conf`：**
```bash
shell> mkdir /etc/kafka && cat << 'EOF' > /etc/kafka/kafka_server_jaas.conf
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin123"
    user_admin="admin123"
    user_guest="guest123";
};
EOF
```
> 配置了两个用户：`admin`和`guest`

**JVM配置`-Djava.security.auth.login.config`**
```bash
shell> vim bin/kafka-server-start.sh
# 增加以下指令补充`java.security.auth.login.config`配置，kafka-run-class.sh 脚本中用到 KAFKA_OPTS 配置
export KAFKA_OPTS="-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
```

**客户端连接配置：**
```properties
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="admin" \
    password="admin123";
```

**Kafka命令行执行之鉴权配置：**
1. 创建鉴权配置文件
```bash
shell> cat << 'EOF' > /etc/kafka/kafka_client.properties
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="admin" \
    password="admin123";

EOF
```
2. 执行命令时带上配置文件: `bin/kafka-{xx}.sh ... --bootstrap-server {ip}:9092 --command-config /etc/kafka/kafka_client.properties`

> 注：如果连接Kafka不需要账号密码，则以上配置可忽略，且下面配置文件中`SASL_PLAINTEXT`替换为`PLAINTEXT`。

#### 3.1 KRaft 模式

**配置Kafka (config/kraft/server.properties)：**

必须修改的配置项：
* `process.roles` - 集群部署时`controller`角色的节点不能小于3
* `node.id` - 所有的`broker`和`controller`节点值必须唯一
  * `192.168.202.245` --> `1`
  * `192.168.202.246` --> `2`
  * `192.168.202.247` --> `3`
* `controller.quorum.voters`
* `listeners` - 修改成对应节点IP
* `advertised.listeners` - 修改成对应节点IP
* `log.dirs`
* `metadata.log.dir`

```properties
#
# This configuration file is intended for use in KRaft mode, where
# Apache ZooKeeper is not present.  See config/kraft/README.md for details.
#

############################# Server Basics #############################

# The role of this server. Setting this puts us in KRaft mode. Possible values: broker|controller|broker,controller
process.roles=broker,controller

# The node id associated with this instance's roles
node.id=1

# The connect string for the controller quorum. All the controllers must be enumerated.
# All of the servers in a Kafka cluster discover the quorum voters using the controller.quorum.voters property.
# Every broker and controller must set the controller.quorum.voters property. The node ID supplied in the 
# controller.quorum.voters property must match the corresponding id on the controller servers.
controller.quorum.voters=1@192.168.202.245:9094,2@192.168.202.246:9094,3@192.168.202.247:9094

# Maximum time in milliseconds to wait without being able to fetch from the leader before triggering a new election
controller.quorum.election.timeout.ms=2000

# Maximum time without a successful fetch from the current leader before becoming a candidate and triggering an election for voters; 
# Maximum time without receiving fetch from a majority of the quorum before asking around to see if there's a new epoch for leader
controller.quorum.fetch.timeout.ms=4000

# The configuration controls the maximum amount of time the client will wait for the response of a request. 
# If the response is not received before the timeout elapses the client will resend the request if necessary or fail the request if retries are exhausted.
controller.quorum.request.timeout.ms=4000

############################# Socket Server Settings #############################

# The address the socket server listens on.
# Combined nodes (i.e. those with `process.roles=broker,controller`) must list the controller listener here at a minimum.
# If the broker listener is not defined, the default listener will use a host name that is equal to the value of java.net.InetAddress.getCanonicalHostName(),
# with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=EXTERNAL://192.168.202.245:9092,INTERNAL://192.168.202.245:9093,CONTROLLER://192.168.202.245:9094

# Name of listener used for communication between brokers.
# The primary purpose of the inter-broker listener is partition replication.
inter.broker.listener.name=INTERNAL

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
advertised.listeners=EXTERNAL://192.168.202.245:9092,INTERNAL://192.168.202.245:9093

# A comma-separated list of the names of the listeners used by the controller.
# If no explicit mapping set in `listener.security.protocol.map`, default will be using PLAINTEXT protocol
# This is required if running in KRaft mode.
controller.listener.names=CONTROLLER

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
listener.security.protocol.map=EXTERNAL:SASL_PLAINTEXT,INTERNAL:SASL_PLAINTEXT,CONTROLLER:SASL_PLAINTEXT

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600

# The number of queued requests allowed for data-plane, before blocking the network threads
queued.max.requests=500

############################# Log Basics #############################

# A comma separated list of directories under which to store log files.
# Kafka should have its own dedicated disk(s) or SSD(s).
log.dirs=/data/kafka/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across the brokers.
num.partitions=8

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

# The default replication factors for automatically created topics
default.replication.factor=2

# When a producer sets acks to "all" (or "-1"), min.insync.replicas specifies the minimum number of replicas 
# that must acknowledge a write for the write to be considered successful. If this minimum cannot be met, 
# then the producer will raise an exception (either NotEnoughReplicas or NotEnoughReplicasAfterAppend).
min.insync.replicas=1

# The maximum time before a new log segment is rolled out (in hours)
log.roll.hours=168

# The number of threads that can move replicas between log directories, which may include disk I/O
#num.replica.alter.log.dirs.threads=

# Number of fetcher threads used to replicate records from each source broker. 
# The total number of fetchers on each broker is bound by num.replica.fetchers multiplied by the number of brokers in the cluster.
# Increasing this value can increase the degree of I/O parallelism in the follower and leader broker at the cost of higher CPU and memory utilization.
num.replica.fetchers=1

############################# KRaft Metadata #############################

# This configuration determines where we put the metadata log for clusters in KRaft mode. 
# If it is not set, the metadata log is placed in the first log directory from log.dirs.
metadata.log.dir=/data/kafka/metadata

# The maximum combined size of the metadata log and snapshots before deleting old snapshots and log files. 
# Since at least one snapshot must exist before any logs can be deleted, this is a soft limit.
#metadata.max.retention.bytes=-1

# The number of milliseconds to keep a metadata log file or snapshot before deleting it. 
# Since at least one snapshot must exist before any logs can be deleted, this is a soft limit.
metadata.max.retention.ms=604800000

############################# Topic  #############################

# Enable auto creation of topic on the server
auto.create.topics.enable=false

# Enables delete topic. Delete topic through the admin tool will have no effect if this config is turned off
delete.topic.enable=true

# The largest record batch size allowed by Kafka (after compression if compression is enabled)
message.max.bytes=1048588

# Indicates whether to enable replicas not in the ISR set to be elected as leader as a last resort, even though doing so may result in data loss
unclean.leader.election.enable=false

# The amount of time to retain delete tombstone markers for log compacted topics. This setting also gives a bound 
# on the time in which a consumer must complete a read if they begin from offset 0 to ensure that 
# they get a valid snapshot of the final stage (otherwise delete tombstones may be collected before they complete their scan).
log.cleaner.delete.retention.ms=86400000

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2

# After a consumer group loses all its consumers (i.e. becomes empty) its offsets will be kept for this retention period before getting discarded. 
# For standalone consumers (using manual assignment), offsets will be expired after the time of last commit plus this retention period.
offsets.retention.minutes=10080

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

# The amount of time to wait before deleting a file from the filesystem
log.segment.delete.delay.ms=60000

############################# Security Settings #############################
# 配置超管
super.users=User:admin

# The list of SASL mechanisms enabled in the Kafka server
sasl.enabled.mechanisms=PLAIN

# SASL mechanism used for inter-broker communication
sasl.mechanism.inter.broker.protocol=PLAIN

# SASL mechanism used for communication with controllers
sasl.mechanism.controller.protocol=PLAIN

```

**格式化存储目录：**
```bash
# 生产一个新的 Cluster ID
shell> bin/kafka-storage.sh random-uuid
# 所有节点格式化存储目录，KAFKA_CLUSTER_ID 由上面的指令产生
shell> bin/kafka-storage.sh format -t {KAFKA_CLUSTER_ID} -c config/kraft/server.properties
```
> 注：所有 Kafka 节点必须使用相同的`KAFKA_CLUSTER_ID`格式化目录

**启动：**
```bash
# `kafka-server-start.sh`脚本中设置 KAFKA_HEAP_OPTS 变量以便 Kafka 使用
# -Xmx -Xms 根据实际情况而定
# export KAFKA_HEAP_OPTS="-Xmx6G -Xms6G -XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80 -XX:+ExplicitGCInvokesConcurrent"
shell> vim bin/kafka-server-start.sh
# Start the Kafka Server
shell> bin/kafka-server-start.sh -daemon config/kraft/server.properties
```

**停止：**
```bash
shell> bin/kafka-server-stop.sh
```

> 注：`kafka-server-start.sh`默认JVM配置为`-Xmx1G -Xms1G`，明显不适用于生产环境。修改Kafka JVM配置，请提供`KAFKA_HEAP_OPTS`环境变量。
> 具体请查看`kafka-server-start.sh`脚本内容。

#### 3.2 ZooKeeper 模式

**配置Zookeeper (config/zookeeper.properties)：**

必须修改配置项：
* `dataDir`
* `dataLogDir`
* `server`

```properties
# 单位：毫秒 - 用于心跳检测。最小的session超时为 tickTime*2
tickTime=2000
# The number of ticks that the initial synchronization phase can take.
# Limit the length of time the ZooKeeper servers in quorum have to connect to a leader
initLimit=10
# The number of ticks that can pass between sending a request and getting an acknowledgement.
# Limits how far out of date a server can be from a leader
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/data/zk/data
# 可选配置，不配置则log写入dataDir目录中，与snapshot存放不同dedicated device会提高性能。
dataLogDir=/data/zk/dataLog
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0

# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080

# 集群配置
server.1=192.168.202.245:2888:3888
server.2=192.168.202.246:2888:3888
server.3=192.168.202.247:2888:3888

# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=8
# Purge task interval in hours. Set to "0" to disable auto purge feature
autopurge.purgeInterval=6
```

**每个ZK节点的dataDir目录新建myid文件：**
```shell
# 192.168.202.245 节点
shell> ZK_DATA_DIR="/data/zk/data" && mkdir -p $ZK_DATA_DIR && echo 1 > $ZK_DATA_DIR/myid
# 192.168.202.246 节点
shell> ZK_DATA_DIR="/data/zk/data" && mkdir -p $ZK_DATA_DIR && echo 2 > $ZK_DATA_DIR/myid
# 192.168.202.247 节点
shell> ZK_DATA_DIR="/data/zk/data" && mkdir -p $ZK_DATA_DIR && echo 3 > $ZK_DATA_DIR/myid
```

**配置Kafka (config/server.properties)：**

必须修改的配置项：
* `broker.id` - 每个节点值必须唯一
  * `192.168.202.245` --> `0`
  * `192.168.202.246` --> `1`
  * `192.168.202.247` --> `2`
* `listeners` - 修改成对应节点IP
* `advertised.listeners` - 修改成对应节点IP
* `log.dirs`
* `zookeeper.connect`

```properties
#
# This configuration file is intended for use in ZK-based mode, where Apache ZooKeeper is required.
# See kafka.server.KafkaConfig for additional details and defaults
#

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
# Start with 0 and increment by 1 for each new broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=EXTERNAL://192.168.202.245:9092,INTERNAL://192.168.202.245:9093,CONTROLLER://192.168.202.245:9094

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
advertised.listeners=EXTERNAL://192.168.202.245:9092,INTERNAL://192.168.202.245:9093,CONTROLLER://192.168.202.245:9094

# Name of listener used for communication between brokers.
# The primary purpose of the inter-broker listener is partition replication.
inter.broker.listener.name=INTERNAL

# Name of listener used for communication between controller and brokers. (Zk模式)
control.plane.listener.name=CONTROLLER

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
listener.security.protocol.map=EXTERNAL:SASL_PLAINTEXT,INTERNAL:SASL_PLAINTEXT,CONTROLLER:SASL_PLAINTEXT

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600

# The number of queued requests allowed for data-plane, before blocking the network threads
queued.max.requests=500

############################# Log Basics #############################

# A comma separated list of directories under which to store log files.
# Kafka should have its own dedicated disk(s) or SSD(s).
log.dirs=/data/kafka/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across the brokers.
num.partitions=8

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

# The default replication factors for automatically created topics
default.replication.factor=2

# When a producer sets acks to "all" (or "-1"), min.insync.replicas specifies the minimum number of replicas 
# that must acknowledge a write for the write to be considered successful. If this minimum cannot be met, 
# then the producer will raise an exception (either NotEnoughReplicas or NotEnoughReplicasAfterAppend).
min.insync.replicas=1

# The maximum time before a new log segment is rolled out (in hours)
log.roll.hours=168

# The number of threads that can move replicas between log directories, which may include disk I/O
#num.replica.alter.log.dirs.threads=

# Number of fetcher threads used to replicate records from each source broker. 
# The total number of fetchers on each broker is bound by num.replica.fetchers multiplied by the number of brokers in the cluster.
# Increasing this value can increase the degree of I/O parallelism in the follower and leader broker at the cost of higher CPU and memory utilization.
num.replica.fetchers=1

############################# Topic  #############################

# Enable auto creation of topic on the server
auto.create.topics.enable=false

# Enables delete topic. Delete topic through the admin tool will have no effect if this config is turned off
delete.topic.enable=true

# The largest record batch size allowed by Kafka (after compression if compression is enabled)
message.max.bytes=1048588

# Indicates whether to enable replicas not in the ISR set to be elected as leader as a last resort, even though doing so may result in data loss
unclean.leader.election.enable=false

# The amount of time to retain delete tombstone markers for log compacted topics. This setting also gives a bound 
# on the time in which a consumer must complete a read if they begin from offset 0 to ensure that 
# they get a valid snapshot of the final stage (otherwise delete tombstones may be collected before they complete their scan).
log.cleaner.delete.retention.ms=86400000

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2

# After a consumer group loses all its consumers (i.e. becomes empty) its offsets will be kept for this retention period before getting discarded. 
# For standalone consumers (using manual assignment), offsets will be expired after the time of last commit plus this retention period.
offsets.retention.minutes=10080

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours. Default -1.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according to the retention policies
log.retention.check.interval.ms=300000

# The amount of time to wait before deleting a file from the filesystem
log.segment.delete.delay.ms=60000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the root directory for all kafka znodes.
zookeeper.connect=192.168.202.245:2181,192.168.202.246:2181,192.168.202.247:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000

# The maximum number of unacknowledged requests the client will send to Zookeeper before blocking.
zookeeper.max.in.flight.requests=10

# Zookeeper session timeout
zookeeper.session.timeout.ms=18000

############################# Security Settings #############################
# 配置超管
super.users=User:admin

# The list of SASL mechanisms enabled in the Kafka server
sasl.enabled.mechanisms=PLAIN

# SASL mechanism used for inter-broker communication
sasl.mechanism.inter.broker.protocol=PLAIN

# SASL mechanism used for communication with controllers
sasl.mechanism.controller.protocol=PLAIN

```


**启动 Zookeeper：**
```bash
# `zookeeper-server-start.sh`脚本中设置 KAFKA_HEAP_OPTS 变量以便 Zookeeper 使用
# export KAFKA_HEAP_OPTS="-Xmx512M -Xms512M"
shell> vim bin/zookeeper-server-start.sh
# Start the ZooKeeper service
shell> bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

**启动 Kafka：**
```bash
# `kafka-server-start.sh`脚本中设置 KAFKA_HEAP_OPTS 变量以便 Kafka 使用
# -Xmx -Xms 根据实际情况而定
# export KAFKA_HEAP_OPTS="-Xmx6G -Xms6G -XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80 -XX:+ExplicitGCInvokesConcurrent"
shell> vim bin/kafka-server-start.sh
# Start the Kafka broker service
shell> bin/kafka-server-start.sh -daemon config/server.properties
```

**停止：**
```bash
shell> bin/zookeeper-server-stop.sh
shell> bin/kafka-server-stop.sh
```

> 注：使用`zookeeper-server-start.sh`启动时默认JVM配置`-Xmx512M -Xms512M`。如想修改ZK的JVM，请在执行`zookeeper-server-start.sh`
> 前提供`KAFKA_HEAP_OPTS`环境变量。具体请查看`zookeeper-server-start.sh`脚本内容。

> 注：`kafka-server-start.sh`默认JVM配置为`-Xmx1G -Xms1G`，明显不适用于生产环境。修改Kafka JVM配置，请提供`KAFKA_HEAP_OPTS`环境变量。
> 具体请查看`kafka-server-start.sh`脚本内容。

#### 3.3 配置文档

##### a. [Broker Configs](https://kafka.apache.org/documentation/#brokerconfigs)

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

**Process Roles（KRaft）**

In KRaft mode each Kafka server can be configured as a controller, a broker, or both using the process.roles property.
This property can have the following values:

* If `process.roles` is set to `broker`, the server acts as a broker.
* If `process.roles` is set to `controller`, the server acts as a controller.
* If `process.roles` is set to `broker,controller`, the server acts as both a broker and a controller.
* If `process.roles` is not set at all, it is assumed to be in `ZooKeeper mode`.

Kafka servers that act as both brokers and controllers are referred to as "combined" servers. Combined servers are simpler
to operate for small use cases like a development environment. The key disadvantage is that the controller will be less isolated
from the rest of the system. For example, it is not possible to roll or scale the controllers separately from the brokers in combined mode. 
if activity on the broker causes an out of memory condition, the controller part of the server is not isolated from that OOM condition.
Combined mode is not recommended in critical deployment environments.

**Controllers（KRaft）**

In KRaft mode, only a small group of specific Kafka servers are selected to be controllers (unlike the ZooKeeper-based mode, where any server can become the Controller). 
The servers selected to be controllers will participate in the metadata quorum. Each controller is either an active 
or a hot standby for the current active controller.

A Kafka admin will typically select 3 or 5 servers for this role, depending on factors like cost and the number of 
concurrent failures your system should withstand without availability impact. A majority of the controllers must be alive 
in order to maintain availability. With 3 controllers, the cluster can tolerate 1 controller failure; with 5 controllers, 
the cluster can tolerate 2 controller failures.

**Quorum Voters（KRaft）**

All nodes in the system must set the `controller.quorum.voters` configuration.
All of the servers in a Kafka cluster discover the quorum voters using the `controller.quorum.voters` property. 
This identifies the quorum controller servers that should be used. All the controllers must be enumerated.
This is similar to how, when using ZooKeeper, the `zookeeper.connect` configuration must contain all the ZooKeeper servers.
Each controller is identified with their `id`, `host` and `port` information. For example:
```properties
controller.quorum.voters=id1@host1:port1,id2@host2:port2,id3@host3:port3
```

So if you have 10 brokers and 3 controllers named controller1, controller2, controller3, you might have the following configuration on controller1:
```properties
process.roles=controller
node.id=1
listeners=CONTROLLER://controller1.example.com:9093
controller.quorum.voters=1@controller1.example.com:9093,2@controller2.example.com:9093,3@controller3.example.com:9093
```

Every broker and controller must set the `controller.quorum.voters` property. The node ID supplied in the 
`controller.quorum.voters` property must match the corresponding id on the controller servers. For example, 
on controller1, node.id must be set to 1, and so forth. Each node ID must be unique across all the servers 
in a particular cluster. No two servers can have the same node ID regardless of their `process.roles` values.

> 注：Controller IDs 没有必要从 0 或 1 开始

##### b. [Topic-Level Configs](https://kafka.apache.org/documentation/#topicconfigs)

> 注：Topic 配置也可以动态修改

##### c. [Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)

##### d. [Consumer Configs](https://kafka.apache.org/documentation/#consumerconfigs)

##### e. [Kafka Connect Configs](https://kafka.apache.org/documentation/#connectconfigs)

* [Source Connector Configs](https://kafka.apache.org/documentation/#sourceconnectconfigs)
* [Sink Connector Configs](https://kafka.apache.org/documentation/#sinkconnectconfigs)

##### f. [Kafka Streams Configs](https://kafka.apache.org/documentation/#streamsconfigs)

##### g. [Admin Configs](https://kafka.apache.org/documentation/#adminclientconfigs)


## 命令

### 查看指令帮助信息
```bash
shell> bin/xxx.sh --help
```

### kafka-acls.sh
This tool helps to manage acls on kafka.
> [文档链接](https://kafka.apache.org/documentation/#security_authz_cli)

### kafka-broker-api-versions.sh
This tool helps to retrieve broker version information.

### kafka-cluster.sh
The Kafka cluster tool.
* `cluster-id` - Get information about the ID of a cluster.
* `unregister` - Unregister a broker.

### kafka-configs.sh
This tool helps to manipulate and describe entity config for a `topic`, `client`, `user`, `broker` or `ip`.

### kafka-console-producer.sh
This tool helps to read data from standard input and publish it to Kafka.

```bash
# 生产事件，标准输入中每行内容产生一个事件，按 `Ctrl-C` 退出
shell> bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server {ip}:9092
```

### kafka-console-consumer.sh
This tool helps to read data from Kafka topics and outputs it to standard output.

```bash
# 读取事件，按 `Ctrl-C` 退出
shell> bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server {ip}:9092
```

### kafka-consumer-groups.sh
This tool helps to list, describe, or delete the consumer groups.

> [文档链接](https://kafka.apache.org/documentation/#basic_ops_consumer_group)

### kafka-consumer-perf-test.sh
This tool helps in performance test for the full zookeeper consumer

### kafka-delegation-tokens.sh
This tool helps to create, renew, expire, or describe delegation tokens.

> [文档链接](https://kafka.apache.org/documentation/#security_delegation_token)

### kafka-delete-records.sh
This tool helps to delete records of the given partitions down to the specified offset.

### kafka-dump-log.sh（KRaft）
This tool helps to parse a log file and dump its contents to the console, useful for debugging a seemingly corrupt log segment.
```bash
# decodes and prints the records in the first log segment
shell> bin/kafka-dump-log.sh --cluster-metadata-decoder --files metadata_log_dir/__cluster_metadata-0/00000000000000000000.log
# decodes and prints the recrods in the a cluster metadata snapshot
shell> bin/kafka-dump-log.sh --cluster-metadata-decoder --files metadata_log_dir/__cluster_metadata-0/00000000000000000100-0000000001.checkpoint
```

> [文档链接](https://kafka.apache.org/documentation/#kraft_dump_log)

### kafka-get-offsets.sh
An interactive shell for getting topic-partition offsets.

> 运行`bin/kafka-get-offsets.sh`会显示帮助信息

### kafka-leader-election.sh
This tool attempts to elect a new leader for a set of topic partitions. The type of elections supported are preferred replicas and unclean replicas.

### kafka-log-dirs.sh
This tool helps to query log directory usage on the specified brokers.

### kafka-metadata-quorum.sh（KRaft）
This tool describes kraft metadata quorum status.
```bash
# 查看集群元数据的运行状态
shell> bin/kafka-metadata-quorum.sh --bootstrap-server  broker_host:port describe --status
# 查看集群元数据的副本信息
shell> bin/kafka-metadata-quorum.sh --bootstrap-server broker_host:port describe --replication
```

### kafka-metadata-shell.sh（KRaft）
The kafka-metadata-shell tool can be used to interactively inspect the state of the cluster metadata partition.
```bash
# interactively inspect the state of the cluster metadata partition
shell> bin/kafka-metadata-shell.sh  --snapshot metadata_log_dir/__cluster_metadata-0/00000000000000000000.log
>> ls /
brokers  local  metadataQuorum  topicIds  topics
>> ls /topics
foo
>> cat /topics/foo/0/data
{
  "partitionId" : 0,
  "topicId" : "5zoAlv-xEh9xRANKXt1Lbg",
  "replicas" : [ 1 ],
  "isr" : [ 1 ],
  "removingReplicas" : null,
  "addingReplicas" : null,
  "leader" : 1,
  "leaderEpoch" : 0,
  "partitionEpoch" : 0
}
>> exit
```

### kafka-mirror-maker.sh
This tool helps to continuously copy data between two Kafka clusters.

### kafka-producer-perf-test.sh
This tool is used to verify the producer performance.

### kafka-reassign-partitions.sh
This tool helps to move topic partitions between replicas.

> [文档链接](https://kafka.apache.org/documentation/#basic_ops_automigrate)

### kafka-replica-verification.sh
Validate that all replicas for a set of topics have the same data.

### kafka-storage.sh
The Kafka storage tool.

```bash
# Get information about the Kafka log directories on this node.
shell> bin/kafka-storage.sh info ...
# Format the Kafka log directories on this node.
# [Replace KRaft Controller Disk](https://kafka.apache.org/documentation/#replace_disk)
shell> bin/kafka-storage.sh format --cluster-id uuid --config server_properties
# Generate a cluster ID for your new cluster. 
# This cluster ID must be used when formatting each server in the cluster with the `kafka-storage.sh format` command.
shell> bin/kafka-storage.sh random-uuid
```
* [Storage Tool](https://kafka.apache.org/documentation/#kraft_storage)

### kafka-streams-application-reset.sh
This tool helps to quickly reset an application in order to reprocess its data from scratch.

* This tool resets offsets of input topics to the earliest available offset (by default), or to a specific defined position and it skips to the end of intermediate topics (topics that are input and output topics, e.g., used by deprecated through() method).
* This tool deletes the internal topics that were created by Kafka Streams (topics starting with "<application.id>-").
  The tool finds these internal topics automatically. If the topics flagged automatically for deletion by the dry-run are unsuitable, you can specify a subset with the "--internal-topics" option.
* This tool will not delete output topics (if you want to delete them, you need to do it yourself with the bin/kafka-topics.sh command).
* This tool will not clean up the local state on the stream application instances (the persisted stores used to cache aggregation results).
  You need to call KafkaStreams#cleanUp() in your application or manually delete them from the directory specified by "state.dir" configuration (${java.io.tmpdir}/kafka-streams/<application.id> by default).
* When long session timeout has been configured, active members could take longer to get expired on the broker thus blocking the reset job to complete. Use the "--force" option could remove those left-over members immediately. Make sure to stop all stream applications when this option is specified to avoid unexpected disruptions.

> Important! You will get wrong output if you don't clean up the local stores after running the reset tool!

> Warning! This tool makes irreversible changes to your application. It is strongly recommended that you run this once with "--dry-run" to preview your changes before making them.

### kafka-topics.sh
This tool helps to create, delete, describe, or change a topic.

```bash
# 创建 topic
shell> bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server {ip}:9092
# 查看 topic
shell> bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server {ip}:9092
# 删除 topic
shell> bin/kafka-topics.sh --bootstrap-server broker_host:port --delete --topic my_topic_name
# 新增 topic 配置
shell> bin/kafka-configs.sh --bootstrap-server broker_host:port --entity-type topics --entity-name my_topic_name --alter --add-config x=y
# 删除 topic 配置
shell> bin/kafka-configs.sh --bootstrap-server broker_host:port --entity-type topics --entity-name my_topic_name --alter --delete-config x
```

### kafka-transactions.sh
This tool is used to analyze the transactional state of  producers  in  the  cluster.  
It can be used to detect and recover from hanging transactions.

### kafka-verifiable-consumer.sh
This tool consumes messages from a specific  topic  and  emits  consumer  events  
(e.g. group rebalances, received messages, and offsets committed) as JSON objects to STDOUT.

### kafka-verifiable-producer.sh
This tool produces increasing integers to  the  specified  topic  and  prints  JSON  metadata  to  stdout on each "send" request, 
making externally visible which messages have been acked and which have not.

### 删除数据
```bash
# ZK 模式
shell> rm -rf /data/kafka/kafka-logs /data/zk/data /data/zk/dataLog
# KRaft 模式
shell> rm -rf /data/kafka/kafka-logs /data/kafka/metadata
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

## Kafka GUI

### Kafka Tool

> [官网链接](https://www.kafkatool.com/) - 客户端程序


### Redpanda Console

```bash
shell> docker run -p 8080:8080 \
  -e KAFKA_BROKERS=192.168.202.245:9092 \
  -e KAFKA_SASL_ENABLED=true \
  -e KAFKA_SASL_MECHANISM=PLAIN \
  -e KAFKA_SASL_USERNAME=admin \
  -e KAFKA_SASL_PASSWORD=admin123 \
  -d docker.redpanda.com/vectorized/console:latest
```

> [官网链接](https://github.com/redpanda-data/console) - Web程序


### Kafka UI

```bash
shell> docker run -p 8080:8080 \
  -e KAFKA_CLUSTERS_0_NAME=local \
  -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=192.168.202.245:9092 \
  -e KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL=SASL_PLAINTEXT \
  -e KAFKA_CLUSTERS_0_PROPERTIES_SASL_MECHANISM=PLAIN \
  -e KAFKA_CLUSTERS_0_PROPERTIES_SASL_JAAS_CONFIG='org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin123";' \
  -d provectuslabs/kafka-ui:latest
```

> [官网链接](https://github.com/provectus/kafka-ui) - Web程序

### EFAK (Eagle For Apache Kafka) - Kafka Eagle

EFAK 提供了 Kafka 监控、告警、管理等相关功能

> [官网链接](https://www.kafka-eagle.org/) - Web程序


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


## 问题

### 1. 客户端高可用问题 - 服务端频繁先后宕机重启，虽然服务端正常，但会导致客户端不能正常功能，一直用之前的服务重连。

原因：客户端应用启动时会根据`bootstrap-servers`配置，选择一个节点和服务端通信，服务端返回有效的节点数据，并选择一个节点作为Coordinator和集群进行通信。
客户端会缓存这个数据5分钟（默认值），通过`metadata.max.age.ms`参数控制。 当Partition leader发生改变、缓存时间过期、Coordinator节点通信失败，
此时会从缓存的节点列表里面挑选其他节点作为Coordinator节点并跟新最新的数据。

以下场景存在问题：当服务端的节点相继shutdown，只有一个节点生存（A）。此时客户端会跟新本地缓存，只保留一个有效节点，且此节点为Coordinator节点。
在5分钟内（缓冲有效时间内）其他节点相继startup，然后再shutdown最后生存的节点A。此时客户端缓存中只保留了A节点，然而A又挂了，所以只能不停地去尝试连接A。
只要A不恢复，客户端应用则无法正常工作。但此时的Kafka集群是正常工作的。

解决方案：
1. 重启客户端应用
2. 服务端失败节点恢复
3. `metadata.max.age.ms`参数值调小（不能根本解决问题）
4. `metadata.recovery.strategy` (新功能，官方实现中)

测试时打开相关日志，并搜索`Updated cluster metadata`关键字查看数据更新情况：
```yaml
logging:
  level:
    org.apache.kafka.clients.Metadata: trace
    org.apache.kafka.clients.consumer.internals.AbstractCoordinator: trace
```

参考地址：
* [A consumer can't discover new group coordinator when the cluster was partly restarted](https://issues.apache.org/jira/browse/KAFKA-8206)
* [Proactively discover alive brokers from bootstrap server lists when all nodes are down](https://issues.apache.org/jira/browse/KAFKA-13653)
* [KAFKA-8206: Allow client to rebootstrap](https://github.com/apache/kafka/pull/13277)
* [KIP-899: Allow producer and consumer clients to rebootstrap](https://cwiki.apache.org/confluence/display/KAFKA/KIP-899%3A+Allow+producer+and+consumer+clients+to+rebootstrap)

### 2. 服务端高可用

* [4.7 Replication](https://kafka.apache.org/documentation/#replication)



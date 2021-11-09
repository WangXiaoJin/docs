# RabbitMQ安装及配置

## CentOS环境安装

### 系统配置

安装完rabbitmq需要修改系统的部分参数，能提高其并发的连接数和队列数。

```
修改fs.file-max：
	1. `echo 6553560 > /proc/sys/fs/file-max` 当前有效，重启无效
	2. 修改`/etc/sysctl.conf`，加入`fs.file-max = 6553560`。重启生效

修改`/etc/security/limits.conf`，加入以下配置，重启生效：
	* soft nofile 65536
	* hard nofile 65536

ulimit -SHn 65536 当前Session有效
```

> 注：fs.file-max值必须大于ulimit值，ulimit的hard值大于等于soft值。

> [Operating System Kernel Limits](https://www.rabbitmq.com/configure.html#kernel-limits)

### 安装

Clustering安装

有三台机器，`hostname`分别为`server1`、`server2`、`server3`，对应的IP分别为`192.168.1.11`、`192.168.1.12`、`192.168.1.13`。

三台机器上增加`/etc/hosts`配置：
```
192.168.1.11 server1
192.168.1.12 server2
192.168.1.13 server3
```

三台机器上安装MQ服务：
```shell
# 创建 rabbitmq-server 仓库
shell> curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
# 创建 erlang 仓库
shell> curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
# 更新所有软件包
shell> yum update -y
# 生成 rabbitmq_erlang、rabbitmq_rabbitmq 仓库元数据缓存
shell> yum -q makecache -y --disablerepo='*' --enablerepo='rabbitmq_erlang' --enablerepo='rabbitmq_rabbitmq-server'
# 安装依赖包
shell> yum install socat logrotate -y
# 安装 erlang、rabbitmq-server
shell> yum install --enablerepo='rabbitmq_erlang' --enablerepo='rabbitmq_rabbitmq-server' erlang rabbitmq-server -y
# 开机启动
shell> systemctl enable rabbitmq-server
# 启动 rabbitmq 服务
shell> systemctl start rabbitmq-server
```

在三台机器上修改配置文件：
```shell
# 修改配置
shell> cat << EOF > /etc/rabbitmq/rabbitmq.conf
cluster_name = mq-test-cluster

cluster_partition_handling = pause_minority

## If a message delivered to a consumer has not been acknowledge before this timer
## triggers the channel will be force closed by the broker. This ensure that
## faultly consumers that never ack will not hold on to messages indefinitely.
consumer_timeout = 1800000
EOF
# 从 server1 上拷贝 .erlang.cookie，确保三台机器的 .erlang.cookie 值相同
shell> scp root@server1:/var/lib/rabbitmq/.erlang.cookie /var/lib/rabbitmq/.erlang.cookie
# 重启服务
shell> systemctl restart rabbitmq-server
```


加入集群，在`server2`、`server3`上执行下述命令：
```shell
# 停止 app 服务
shell> rabbitmqctl stop_app
# 重置node，删除所有的资源及数据
shell> rabbitmqctl reset
# 加入 server1 集群
shell> rabbitmqctl join_cluster rabbit@server1
# 启动 app 服务
shell> rabbitmqctl start_app
```

创建admin账号（任意挑选一台执行）：
```shell
# 新建用户 - 用户名：admin 密码：admin123
shell> rabbitmqctl add_user admin admin123
# admin 用户设置用户标签为 administrator。management UI访问权限通过用户标签控制。参考链接：https://www.rabbitmq.com/management.html#permissions
shell> rabbitmqctl set_user_tags admin administrator
# 设置 admin 在 "/" virtual host 中的 configure、write、read 权限
shell> rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

三台机器上安装 management ui 插件，不一定每台机器都需要安装。多台安装后，可在前面套上反向代理，达到负载均衡作用：
```shell
# 安装 management ui 插件
shell> rabbitmq-plugins enable rabbitmq_management
```

* [Production Checklist](https://www.rabbitmq.com/production-checklist.html) 
  * Virtual Hosts, Users, Permissions
  * `Memory` - 默认值 `vm_memory_high_watermark.relative = 0.4`
  * `Disk Space` - 推荐值 `disk_free_limit.relative = 1.5`
  * Open File Handles Limit
  * Cluster Size
  * Node Time Synchronization

> 注：rabbitmq默认是用`rabbitmq`用户启动服务的，所以如果你改变了Mnesia database或logs文件的地址，请确保这些文件的拥有者是`rabbitmq`，让其有权限操作文件。

### Distributed RabbitMQ

* [Distributed RabbitMQ](https://www.rabbitmq.com/distributed.html)
* [`Federation` VS `Shovel` VS `Clustering`](https://www.rabbitmq.com/distributed.html#summary)
  
#### Clustering

_Node Names (Identifiers)_

RabbitMQ nodes are identified by node names. A node name consists of two parts, a prefix (usually `rabbit`) and hostname. 
For example, `rabbit@node1.messaging.svc.local` is a node name with the prefix of `rabbit` and hostname of `node1.messaging.svc.local`.

Node names in a cluster must be unique. If more than one node is running on a given host (this is usually the case in 
development and QA environments), they must use different prefixes, e.g. `rabbit1@hostname` and `rabbit2@hostname`.

In a cluster, nodes identify and contact each other using node names. This means that the hostname part of every node name [must resolve](https://www.rabbitmq.com/clustering.html#hostname-resolution-requirement). 
[CLI tools](https://www.rabbitmq.com/cli.html) also identify and address nodes using node names.

When a node starts up, it checks whether it has been assigned a node name. This is done via the `RABBITMQ_NODENAME`
[environment variable](https://www.rabbitmq.com/configure.html#supported-environment-variables). If no value was explicitly configured, 
the node resolves its hostname and prepends `rabbit` to it to compute its node name.

If a system uses fully qualified domain names (FQDNs) for hostnames, RabbitMQ nodes and CLI tools must be configured to 
use so called long node names. For server nodes this is done by setting the `RABBITMQ_USE_LONGNAME` environment variable to `true`.

For CLI tools, either `RABBITMQ_USE_LONGNAME` must be set or the `--longnames` option must be specified.


_What is Replicated?_

All data/state required for the operation of a RabbitMQ broker is replicated across all nodes. An exception to this are message queues, 
which by default reside on one node, though they are visible and reachable from all nodes. To replicate queues across nodes in a cluster, 
use a queue type that supports replication. This topic is covered in the [Quorum Queues](https://www.rabbitmq.com/quorum-queues.html) guide.


_Nodes are Equal Peers_

Some distributed systems have leader and follower nodes. This is generally not true for RabbitMQ. All nodes in a RabbitMQ 
cluster are equal peers: there are no special nodes in RabbitMQ core. This topic becomes more nuanced when [quorum queues](https://www.rabbitmq.com/quorum-queues.html) 
and plugins are taken into consideration but for most intents and purposes, all cluster nodes should be considered equal.

Many [CLI tool](https://www.rabbitmq.com/cli.html) operations can be executed against any node. An [HTTP API](https://www.rabbitmq.com/management.html) client can target any cluster node.

Individual plugins can designate (elect) certain nodes to be "special" for a period of time. For example, [federation links](https://www.rabbitmq.com/federation.html) 
are colocated on a particular cluster node. Should that node fail, the links will be restarted on a different node.

In versions older than 3.6.7, [RabbitMQ management plugin](https://www.rabbitmq.com/management.html) used a dedicated node for stats collection and aggregation.


_Restarting Cluster Nodes_

A restarted node will sync the schema and other information from its peers on boot. Before this process completes, 
the node **won't be fully started and functional**.

A stopping node picks an online cluster member (only disc nodes will be considered) to sync with after restart. 
Upon restart the node will try to contact that peer 10 times by default, with 30 second response timeouts.

In case the peer becomes available in that time interval, the node successfully starts, syncs what it needs from the peer and keeps going.

If the peer does not become available, the restarted node will **give up and voluntarily stop**. Such condition can be 
identified by the timeout (`timeout_waiting_for_tables`) warning messages in the logs that eventually lead to node startup failure:

When a node has no online peers during shutdown, it will start without attempts to sync with any known peers. 
It does not start as a standalone node, however, and peers will be able to rejoin it.

When the entire cluster is brought down therefore, the last node to go down is the only one that didn't have any running 
peers at the time of shutdown. That node can start without contacting any peers first. Since nodes will try to contact 
a known peer for up to 5 minutes (by default), nodes can be restarted in any order in that period of time. In this case 
they will rejoin each other one by one successfully. This window of time can be adjusted using two configuration settings:

```properties
# wait for 60 seconds instead of 30
mnesia_table_loading_retry_timeout = 60000

# retry 15 times instead of 10
mnesia_table_loading_retry_limit = 15
```

By adjusting these settings and tweaking the time window in which known peer has to come back it is possible to account 
for cluster-wide redeployment scenarios that can be longer than 5 minutes to complete.

During upgrades, sometimes the last node to stop must be the first node to be started after the upgrade. 
That node will be designated to perform a cluster-wide schema migration that other nodes can sync from and apply when they rejoin.


_What is Queue Mirroring_

`Important`: mirroring of classic queues will be `removed in a future version` of RabbitMQ. Consider using [quorum queues](https://www.rabbitmq.com/quorum-queues.html) 
or a non-replicated classic queue instead.

By default, contents of a queue within a RabbitMQ cluster are located on a single node (the node on which the queue was declared). 
This is in contrast to exchanges and bindings, which can always be considered to be on all nodes. Queues can optionally 
run mirrors (additional replicas) on other cluster nodes.

Each mirrored queue consists of one `leader replica` and one or more `mirrors` (replicas). The leader is hosted on one node 
commonly referred as the leader node for that queue. Each queue has its own leader node. All operations for a given queue 
are first applied on the queue's leader node and then propagated to mirrors. This involves enqueueing publishes, 
delivering messages to consumers, tracking [acknowledgements from consumers](https://www.rabbitmq.com/confirms.html) and so on.

Queue mirroring implies a cluster of nodes. It is therefore not recommended for use across a WAN (though of course, 
clients can still connect from as near and as far as needed).

Messages published to the queue are replicated to all mirrors. Consumers are connected to the leader regardless of 
which node they connect to, with mirrors dropping messages that have been acknowledged at the leader. Queue mirroring 
therefore enhances availability, but does not distribute load across nodes (all participating nodes each do all the work).

If the node that hosts queue leader fails, the oldest mirror will be promoted to the new leader as long as it's synchronised. 
[Unsynchronised mirrors](https://www.rabbitmq.com/ha.html#unsynchronised-mirrors) can be promoted, too, depending on queue mirroring parameters.

* [Clustering Guide](https://www.rabbitmq.com/clustering.html)
  * [Node Names (Identifiers)](https://www.rabbitmq.com/clustering.html#node-names)
  * [Hostname Resolution](https://www.rabbitmq.com/clustering.html#hostname-resolution-requirement)
  * [How CLI Tools Authenticate to Nodes (and Nodes to Each Other): the Erlang Cookie](https://www.rabbitmq.com/clustering.html#erlang-cookie)
  * [Cookie File Locations](https://www.rabbitmq.com/clustering.html#cookie-file-locations)
  * [Creating a Cluster](https://www.rabbitmq.com/clustering.html#creating)
  * [Restarting Cluster Nodes](https://www.rabbitmq.com/clustering.html#restarting)
  * [Restarts and Health Checks (Readiness Probes)](https://www.rabbitmq.com/clustering.html#restarting-readiness-probes)
  * [Hostname Changes Between Restarts](https://www.rabbitmq.com/clustering.html#restarting-with-hostname-changes)
  * [Forcing Node Boot in Case of Unavailable Peers](https://www.rabbitmq.com/clustering.html#forced-boot)
  * [Remove nodes from a cluster](https://www.rabbitmq.com/clustering.html#removing-nodes)
  * [Disk and RAM Nodes](https://www.rabbitmq.com/clustering.html#cluster-node-types)

* [Cluster Formation and Peer Discovery](https://www.rabbitmq.com/cluster-formation.html)
  * [How Peer Discovery Works](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-how-does-it-work)
  * [Nodes Rejoining Their Existing Cluster](https://www.rabbitmq.com/cluster-formation.html#rejoining)
  * Peer Discovery
    * [Config File Peer Discovery Backend](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-classic-config)
    * [DNS Peer Discovery Backend](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-dns)
    * [Peer Discovery on AWS (EC2)](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-aws)
    * [Peer Discovery on Kubernetes](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-k8s)
    * [Peer Discovery Using Consul](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-consul)
    * [Peer Discovery Using Etcd](https://www.rabbitmq.com/cluster-formation.html#peer-discovery-etcd)
  * [Race Conditions During Initial Cluster Formation](https://www.rabbitmq.com/cluster-formation.html#initial-formation-race-condition)
  * [Node Health Checks and Forced Removal](https://www.rabbitmq.com/cluster-formation.html#node-health-checks-and-cleanup)
  * [Peer Discovery Failures and Retries](https://www.rabbitmq.com/cluster-formation.html#discovery-retries)

* [Classic Queue Mirroring](https://www.rabbitmq.com/ha.html)
  * [What is Queue Mirroring](https://www.rabbitmq.com/ha.html#what-is-mirroring)
  * [Queue Leader Location](https://www.rabbitmq.com/ha.html#queue-leader-location)
  * ["nodes" Policy and Migrating Leaders](https://www.rabbitmq.com/ha.html#fixed-leader-promotion)
  * [Mirroring of Exclusive Queues](https://www.rabbitmq.com/ha.html#exclusive-queues-are-not-mirrored)
  * [Non-mirrored Queue Behavior in a Cluster](https://www.rabbitmq.com/ha.html#non-mirrored-queue-behavior-on-node-failure)
  * [Mirrored Queue Implementation and Semantics](https://www.rabbitmq.com/ha.html#behaviour)
  * [Publisher Confirms and Transactions](https://www.rabbitmq.com/ha.html#confirms-transactions)
  * [Flow Control](https://www.rabbitmq.com/ha.html#flow-control)
  * [Leader Failures and Consumer Cancellation](https://www.rabbitmq.com/ha.html#cancellation)
  * [Unsynchronised Mirrors](https://www.rabbitmq.com/ha.html#unsynchronised-mirrors)
  * [Promotion of Unsynchronised Mirrors on Failure](https://www.rabbitmq.com/ha.html#promoting-unsynchronised-mirrors)
  * [Stopping Nodes and Synchronisation](https://www.rabbitmq.com/ha.html#start-stop)
  * [Stopping Nodes Hosting Queue Leader with Only Unsynchronised Mirrors](https://www.rabbitmq.com/ha.html#cluster-shutdown)
  * [Loss of a Leader While All Mirrors are Stopped](https://www.rabbitmq.com/ha.html#promotion-while-down)
  * [Batch Synchronization](https://www.rabbitmq.com/ha.html#batch-sync)

* [Clustering and Network Partitions](https://www.rabbitmq.com/partitions.html)
  * Partitions Caused by Suspend and Resume
  * Recovering From a Split-Brain
  * Partition Handling Strategies

* [Net Tick Time (Inter-node Communication Heartbeats)](https://www.rabbitmq.com/nettick.html)


#### Federation

* [Federation Plugin](https://www.rabbitmq.com/federation.html)
* [Federated Exchanges](https://www.rabbitmq.com/federated-exchanges.html)
* [Federated Queues](https://www.rabbitmq.com/federated-queues.html)
* [Federation Reference](https://www.rabbitmq.com/federation-reference.html)

#### Shovel plugin

* [Shovel Plugin](https://www.rabbitmq.com/shovel.html)
* [Static Shovels](https://www.rabbitmq.com/shovel-static.html)
* [Dynamic Shovels](https://www.rabbitmq.com/shovel-dynamic.html)

#### High availability with Pacemaker and DRBD - [参考文档](https://www.rabbitmq.com/pacemaker.html)

* DRBD
* Pacemaker
* CoroSync
* Heartbeat
* OpenAIS
	
### 端口号
* `4369`: [epmd](http://erlang.org/doc/man/epmd.html), a peer discovery service used by RabbitMQ nodes and CLI tools
* `5672`, `5671`: used by AMQP 0-9-1 and 1.0 clients without and with TLS
* `5552`, `5551`: used by the RabbitMQ Stream protocol clients without and with TLS
* `25672`: used for inter-node and CLI tools communication (Erlang distribution server port) and is allocated from a dynamic
  range (limited to a single port by default, computed as AMQP port + 20000). Unless external connections on these ports
  are really necessary (e.g. the cluster uses [federation](https://www.rabbitmq.com/federation.html) or CLI tools are used on machines outside the subnet),
  these ports should not be publicly exposed. See [networking guide](https://www.rabbitmq.com/networking.html) for details.
* `35672-35682`: used by CLI tools (Erlang distribution client ports) for communication with nodes and is allocated
  from a dynamic range (computed as server distribution port + 10000 through server distribution port + 10010).
  See [networking guide](https://www.rabbitmq.com/networking.html) for details.
* `15672`: [HTTP API](https://www.rabbitmq.com/management.html) clients, [management UI](https://www.rabbitmq.com/management.html)
  and [rabbitmqadmin](https://www.rabbitmq.com/management-cli.html) (only if the [management plugin](https://www.rabbitmq.com/management.html) is enabled)
* `61613`, `61614`: [STOMP clients](https://stomp.github.io/stomp-specification-1.2.html) without and with TLS (only if the [STOMP plugin](https://www.rabbitmq.com/stomp.html) is enabled)
* `1883`, `8883`: [MQTT clients](http://mqtt.org/) without and with TLS, if the [MQTT plugin](https://www.rabbitmq.com/mqtt.html) is enabled
* `15674`: STOMP-over-WebSockets clients (only if the [Web STOMP plugin](https://www.rabbitmq.com/web-stomp.html) is enabled)
* `15675`: MQTT-over-WebSockets clients (only if the [Web MQTT plugin](https://www.rabbitmq.com/web-mqtt.html) is enabled)
* `15692`: Prometheus metrics (only if the [Prometheus plugin](https://www.rabbitmq.com/prometheus.html) is enabled)

### 安装参考文档
* [Installing on RPM-based Linux (RedHat Enterprise Linux, CentOS, Fedora, openSUSE)](https://www.rabbitmq.com/install-rpm.html)
* [RabbitMQ and Erlang/OTP Compatibility Matrix](https://www.rabbitmq.com/which-erlang.html)
* [Signed Packages](https://www.rabbitmq.com/signatures.html)

## 维护

```shell
# 查看服务状态
shell> sudo rabbitmqctl status

# checks if the local node is running and CLI tools can successfully authenticate with it
shell> sudo rabbitmq-diagnostics ping

# prints enabled components (applications), TCP listeners, memory usage breakdown, alarms and so on
shell> sudo rabbitmq-diagnostics status

# prints cluster membership information
shell> sudo rabbitmq-diagnostics cluster_status

# prints effective node configuration
shell> sudo rabbitmq-diagnostics environment

shell> sudo rabbitmq-diagnostics listeners
```

* [Command Line Tools](https://www.rabbitmq.com/cli.html)
* [Manual Pages](https://www.rabbitmq.com/manpages.html) - CLI工具参考手册
  * [rabbitmq-env.conf](https://www.rabbitmq.com/rabbitmq-env.conf.5.html): environment variable-based [configuration](https://www.rabbitmq.com/configure.html)
  * [rabbitmqctl](https://www.rabbitmq.com/rabbitmqctl.8.html): service management and [general operator commands](https://www.rabbitmq.com/cli.html)
  * [rabbitmq-diagnostics](https://www.rabbitmq.com/rabbitmq-diagnostics.8.html): commands useful for diagnostics, [health checking](https://www.rabbitmq.com/monitoring.html), and observing system state
  * [rabbitmq-plugins](https://www.rabbitmq.com/rabbitmq-plugins.8.html): [plugin management](https://www.rabbitmq.com/plugins.html)
  * [rabbitmq-upgrade](https://www.rabbitmq.com/rabbitmq-upgrade.8.html): operational tasks related to upgrades
  * [rabbitmq-queues](https://www.rabbitmq.com/rabbitmq-queues.8.html): operational tasks for queues
  * [rabbitmq-server](https://www.rabbitmq.com/rabbitmq-server.8.html): starts a RabbitMQ server node
  * [rabbitmq-service](https://www.rabbitmq.com/rabbitmq-service.8.html): Windows service management
  * [rabbitmq-echopid](https://www.rabbitmq.com/rabbitmq-echopid.8.html): a Windows-specific utility tool

## 参考文档

> [官方文档](https://www.rabbitmq.com/documentation.html)

### Connections / Channels / Virtual Hosts / Consumers

* [Connections](https://www.rabbitmq.com/connections.html)
* [Channels](https://www.rabbitmq.com/channels.html)
* [Virtual Hosts](https://www.rabbitmq.com/vhosts.html)
* [Queues](https://www.rabbitmq.com/queues.html)
* [Publishers](https://www.rabbitmq.com/publishers.html)
  * `Temporarily Blocking Publishing`
* [Consumers](https://www.rabbitmq.com/consumers.html)
  * `Delivery Acknowledgement Timeout` - If a consumer does not ack its delivery for more than the timeout value (`30 minutes by default`), 
  its channel will be closed with a `PRECONDITION_FAILED` channel exception. The error will be logged by the node that the consumer was connected to.
  * `Exclusivity`
  * `Single Active Consumer`
  * `Priority`

### 配置

* [How to Find Config File Location](https://www.rabbitmq.com/configure.html#verify-configuration-config-file-location)
* [Location of rabbitmq.conf, advanced.config and rabbitmq-env.conf](https://www.rabbitmq.com/configure.html#config-location)
* [`rabbitmq.conf` 配置示例](https://github.com/rabbitmq/rabbitmq-server/blob/v3.8.x/deps/rabbit/docs/rabbitmq.conf.example)
* [`advanced.config` 配置示例](https://github.com/rabbitmq/rabbitmq-server/blob/v3.8.x/deps/rabbit/docs/advanced.config.example)
* [How to Inspect and Verify Effective Configuration of a Running Node](https://www.rabbitmq.com/configure.html#verify-configuration-effective-configuration)
* [Configuration Value Encryption](https://www.rabbitmq.com/configure.html#configuration-encryption)
* [Configuration Using Environment Variables](https://www.rabbitmq.com/configure.html#customise-environment)
* [File and Directory Locations](https://www.rabbitmq.com/relocate.html)
* [Logging](https://www.rabbitmq.com/logging.html)
  * `Logged Events`
* [Persistence Configuration](https://www.rabbitmq.com/persistence-conf.html)
* [Networking](https://www.rabbitmq.com/networking.html)
  * How to Temporarily Stop New Client Connections
  * Tuning for a large number of concurrent connections
  * High connection churn scenarios and resource exhaustion
  * TCP buffer size
  * Kernel TCP settings and limits (e.g. TCP keepalives and open file handle limit)
  > [Troubleshooting Network Connectivity](https://www.rabbitmq.com/troubleshooting-networking.html)
* [Parameters and Policies](https://www.rabbitmq.com/parameters.html)
  * If they need to be the same across all nodes in a cluster
  * If they are likely to change at run time
* [Credentials and Passwords](https://www.rabbitmq.com/passwords.html)

#### What is `EPMD` and How is It Used?

[epmd](http://www.erlang.org/doc/man/epmd.html) (for Erlang Port Mapping Daemon) is a small additional daemon that runs
alongside every RabbitMQ node and is used by the [runtime](https://www.rabbitmq.com/runtime.html) to discover what port
a particular node listens on for inter-node communication. The port is then used by peer nodes and [CLI tools](https://www.rabbitmq.com/cli.html).

When a node or CLI tool needs to contact node `rabbit@hostname2` it will do the following:

* Resolve `hostname2` to an IPv4 or IPv6 address using the standard OS resolver or a custom one specified in the [inetrc file](http://erlang.org/doc/apps/erts/inet_cfg.html)
* Contact `epmd` running on `hostname2` using the above address
* Ask `epmd` for the port used by node `rabbit` on it
* Connect to the node using the resolved IP address and the discovered port
* Proceed with communication

> [EPMD 参考文档](https://www.rabbitmq.com/networking.html#epmd)

### Management UI / 认证授权 / 监控 / Alarms / TLS / 排障

#### Management UI

Any cluster node with `rabbitmq-management` plugin enabled can be used for management UI access or data collection by 
monitoring tools. It will reach out to other nodes and collect their stats, then aggregate and return a response to the client.

When monitoring a cluster of nodes, there is `no need` to contact `each node` via HTTP API individually. Instead, contact 
a random node or a load balancer that sits in front of the cluster.

Disabling the metrics collection is the preferred option if it is being used with an external monitoring system, 
as this reduced the overhead that statistics collection and aggregation causes in the broker.
```properties
management_agent.disable_metrics_collector = true
```

* [Management Plugin - 参考文档](https://www.rabbitmq.com/management.html)
* [Access and Permissions](https://www.rabbitmq.com/management.html#permissions)
* [Management Command Line Tool](https://www.rabbitmq.com/management-cli.html)

#### 认证授权

* [Authentication, Authorisation, Access Control](https://www.rabbitmq.com/access-control.html)
  * Default Virtual Host and User
  * "guest" user can only connect from localhost
  * Managing Users and Permissions
  * Authorisation: How Permissions Work
  * User Tags and Management UI Access
  * Built-in Authentication Mechanisms
* [LDAP Support](https://www.rabbitmq.com/ldap.html)

> RabbitMQ may `cache` the results of access control checks on a per-connection or per-channel basis. Hence changes to 
user permissions may only take effect when the `user reconnects`.

#### 监控 / Internal Event Exchange

* [Monitoring](https://www.rabbitmq.com/monitoring.html)
  * Metrics
  * Health Checks
  * Monitoring Tools
* [Internal Event Exchange](https://www.rabbitmq.com/event-exchange.html) - 记录事件
* [Firehose (Message Tracing)](https://www.rabbitmq.com/firehose.html) - 可以跟踪消息的发送和投递


#### Alarms

* [Memory and Disk Alarms](https://www.rabbitmq.com/alarms.html)
  * [Memory Alarms](https://www.rabbitmq.com/memory.html)
    * Configuring the Memory Threshold
    * Configuring the Paging Threshold
  * [Free Disk Space Alarms](https://www.rabbitmq.com/disk-alarms.html)
  * [Reasoning About Memory Use](https://www.rabbitmq.com/memory-use.html)
  * [Flow Control](https://www.rabbitmq.com/flow-control.html)

#### TLS Support

* [TLS Support](https://www.rabbitmq.com/ssl.html)
* [Troubleshooting TLS-enabled Connections](https://www.rabbitmq.com/troubleshooting-ssl.html)
* [Securing Cluster (Inter-node) and CLI Tool Communication with TLS (SSL)](https://www.rabbitmq.com/clustering-ssl.html)

#### 排障

* [Troubleshooting Guidance](https://www.rabbitmq.com/troubleshooting.html)
  * [Runtime Crash Dump Files](https://www.rabbitmq.com/troubleshooting.html#crash-dumps)

### 升级 / 备份 / 恢复

* [Upgrading RabbitMQ](https://www.rabbitmq.com/upgrade.html)

* [Backup and restore](https://www.rabbitmq.com/backup.html)

* [Schema Definition Export and Import](https://www.rabbitmq.com/definitions.html)

* [Blue-green deployment-based upgrade](https://www.rabbitmq.com/blue-green-upgrade.html)

* [Feature Flags](https://www.rabbitmq.com/feature-flags.html)

  Feature flags are a mechanism that controls what features are considered to be enabled or available on all cluster nodes.

  This subsystem was introduced in RabbitMQ 3.8.0 to allow rolling upgrades of cluster members without shutting down the entire cluster.

### Blue-Green Deployment

* [Upgrading RabbitMQ Using Blue-Green Deployment Strategy](https://www.rabbitmq.com/blue-green-upgrade.html)
* [Blue-Green Application Deployments with RabbitMQ](https://tanzu.vmware.com/content/blog/blue-green-application-deployments-with-rabbitmq)

### 协议

#### `AMQP 0-9-1 and extensions` - [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

Queues properties:
* Name
* Durable (the queue will survive a broker restart)
* Exclusive (used by only one connection and the queue will be deleted when that connection closes)
* Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes)
* Arguments (optional; used by plugins and broker-specific features such as message TTL, queue length limit, etc)

Before a queue can be used it has to be declared. Declaring a queue will cause it to be created if it does not already exist. 
The declaration will have no effect if the queue does already exist and its attributes are the same as those in the declaration.
When the existing queue attributes are not the same as those in the declaration a channel-level exception with code 406 (`PRECONDITION_FAILED`) will be raised.

_Queue Names_

Queue names may be up to `255 bytes` of UTF-8 characters. An AMQP 0-9-1 broker can generate a unique queue name on 
behalf of an app. To use this feature, pass an empty string as the queue name argument. The generated name will be returned
to the client with queue declaration response.

Queue names starting with `"amq."` are reserved for internal use by the broker. Attempts to declare a queue with a name 
that violates this rule will result in a channel-level exception with reply code 403 (`ACCESS_REFUSED`).

_Consumers_

* Subscribe to have messages delivered to them ("push API"): this is the recommended option
* Polling ("pull API"): this way is `highly inefficient` and `should be avoided` in most cases

Each consumer (subscription) has an identifier called a `consumer tag`. It can be used to unsubscribe from messages. Consumer tags are just strings.

_Message ordering guarantees_

Messages can be returned to the queue using AMQP methods that feature a requeue parameter (`basic.recover`, `basic.reject`
and `basic.nack`), or due to a channel closing while holding unacknowledged messages. Any of these scenarios caused messages 
to be requeued at the back of the queue for RabbitMQ releases earlier than 2.7.0. From RabbitMQ release 2.7.0, 
messages are always held in the queue in publication order, even in the presence of requeueing or channel closure.

With release 2.7.0 and later it is still possible for individual consumers to observe messages out of order if the 
queue has multiple subscribers. This is due to the actions of other subscribers who may requeue messages. 
From the perspective of the queue the messages are always held in the publication order.

* [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html)
* [AMQP 0-9-1 Complete Reference Guide](https://www.rabbitmq.com/amqp-0-9-1-reference.html)
* [AMQP 0-9-1 Quick Reference](https://www.rabbitmq.com/amqp-0-9-1-quickref.html)
* [AMQP 0-9-1 Protocol Specification](https://www.rabbitmq.com/protocol.html)
  * [Specification - PDF](https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf)
  * [Generated Doc - PDF](https://www.rabbitmq.com/resources/specs/amqp-xml-doc0-9-1.pdf)
* [Broker Semantics](https://www.rabbitmq.com/semantics.html)
* [Compatibility and Conformance](https://www.rabbitmq.com/specification.html)
* [AMQP 0-9-1 Errata](https://www.rabbitmq.com/amqp-0-9-1-errata.html) - 解释规范中错误及模棱两可的内容

#### `STOMP`

* [STOMP - 官网](http://stomp.github.io/)
* [STOMP Plugin](https://www.rabbitmq.com/stomp.html)
* [STOMP-over-WebSockets](https://www.rabbitmq.com/web-stomp.html)

#### `MQTT`

* [MQTT Plugin](https://www.rabbitmq.com/mqtt.html)
* [MQTT-over-WebSockets](https://www.rabbitmq.com/web-mqtt.html)

#### `AMQP 1.0`

#### `HTTP and WebSockets`

> [ Which protocols does RabbitMQ support? ](https://www.rabbitmq.com/protocols.html)

### Reliability / Publisher Confirms

`Publisher confirms` are a RabbitMQ extension to implement reliable publishing.

* [Reliable Delivery](https://www.rabbitmq.com/reliability.html)
* [Publisher Confirms 示例](https://www.rabbitmq.com/tutorials/tutorial-seven-java.html)

### Streams / Lazy Queues / Quorum Queues

#### Streams

Streams are a `new` persistent and replicated data structure in `RabbitMQ 3.9` which models an `append-only log` with non-destructive consumer semantics. 

使用场景：
* Large fan-outs
* Replay / Time-travelling
* Throughput Performance
* Large logs

Streams replicate data across multiple nodes and publisher confirms are only issued once the data has been replicated to a `quorum` of stream replicas.

Streams always store data on disk, however, they do not explicitly flush (fsync) the data from the operating system page 
cache to the underlying storage medium, instead they rely on the operating system to do as and when required.

If more data safety is required then consider using quorum queues instead as no publisher confirms are issued until at 
least a quorum of nodes have both written and flushed the data to disk.

* [Streams 文档](https://www.rabbitmq.com/streams.html)
* [Stream Plugin](https://www.rabbitmq.com/stream.html)

#### Lazy Queues

_Node Startup_

When a node is running and under normal operation, lazy queues will keep all messages on disk, the only exception being in-flight messages.

When a RabbitMQ node starts, all queues, including the lazy ones, will load up to `16,384` messages into RAM. 
If [queue index embedding](https://www.rabbitmq.com/persistence-conf.html) is enabled 
(the `queue_index_embed_msgs_below` configuration parameter is greater than 0), the payloads of those messages will be loaded into RAM as well.

For example, a lazy queue with `20,000` messages of `4,000` bytes each, will load `16,384` messages into memory. 
These messages will use `63MB` of system memory. The queue process will use another `8.4MB` of system memory, bringing the total to just over `70MB`.

This is an important consideration for capacity planning if the RabbitMQ node is memory constrained, or if there are many lazy queues hosted on the node.

`It is important to remember that an under-provisioned RabbitMQ node in terms of memory or disk space will fail to start.`

Setting `queue_index_embed_msgs_below` to `0` will disable payload embedding in the queue index. As a result, 
lazy queues will not load message payloads into memory on node startup. See the Persistence Configuration guide for details.

When `setting queue_index_embed_msgs_below` to `0` all messages will be stored to the message store. With many messages 
across many lazy queues, that can lead to higher disk usage and also higher file descriptor usage.

Message store is append-oriented and uses a compaction mechanism to reclaim disk space. In extreme scenarios it can use two 
times more disk space compared to the sum of message payloads stored on disk. It is important to overprovision disk space 
to account for such peaks.

All messages in the message store are stored in 16MB files called segment files or segments. Each queue has its own 
file descriptor for each segment file it has to access. For example, if 100 queues store 10GB worth of messages, 
there will be 640 files in the message store and up to 64000 file descriptors. Make sure the nodes have a high enough 
[open file limit](https://www.rabbitmq.com/production-checklist.html#resource-limits-file-handle-limit) 
and overprovision it when in doubt (e.g. to 300K or 500K). For new installations it is possible to 
increase file size used by the message store using `msg_store_file_size_limit` configuration key. `Never change segment 
file size for existing installations` as that can result in a subset of messages being ignored by the node and can break segment file compaction.

* [Lazy Queues](https://www.rabbitmq.com/lazy-queues.html)

#### Quorum Queues

`Quorum Queues` 使用 [Raft Consensus Algorithm](https://raft.github.io/) 复制 FIFO Queue。

Performance tails off quite a bit for quorum queue node sizes `larger than 5`. We do not recommend running quorum queues 
`on more than 7` RabbitMQ nodes. The `default quorum queue size is 3` and is controllable using the `x-quorum-initial-group-size` queue argument.

Quorum queues are designed to provide data safety under network partition and failure scenarios. A message that was 
successfully confirmed back to the publisher using the [publisher confirms](https://www.rabbitmq.com/confirms.html) feature should not be lost as long as 
at least a majority of RabbitMQ nodes hosting the quorum queue are not permanently made unavailable.

* [Quorum Queues](https://www.rabbitmq.com/quorum-queues.html)
  * [Poison Message Handling](https://www.rabbitmq.com/quorum-queues.html#poison-message-handling)

### RabbitMQ 对 AMQP 0-9-1 协议的扩展功能

#### Publishing

* [Consumer Acknowledgements and Publisher Confirms](https://www.rabbitmq.com/confirms.html) (aka Publisher Acknowledgements) are a lightweight way to know when RabbitMQ has taken responsibility for messages.
* [Blocked Connection Notifications](https://www.rabbitmq.com/connection-blocked.html) allows clients to be notified when a connection is blocked and unblocked.

#### Consuming

* [Consumer Cancellation Notifications](https://www.rabbitmq.com/consumer-cancel.html) let a consumer know if it has been cancelled by the server.
* [Consumer Prefetch](https://www.rabbitmq.com/consumer-prefetch.html)
* [basic.nack](https://www.rabbitmq.com/nack.html) extends `basic.reject` to support rejecting multiple messages at once.
* [Consumer Priorities](https://www.rabbitmq.com/consumer-priority.html) allow you to send messages to higher priority consumers first.
* [Direct reply-to](https://www.rabbitmq.com/direct-reply-to.html) allows RPC clients to receive replies to their queries without needing to declare a temporary queue.


#### Message Routing

* [Exchange to Exchange Bindings](https://www.rabbitmq.com/e2e.html) allow messages to pass through multiple exchanges for more flexible routing.
* [Alternate Exchanges](https://www.rabbitmq.com/ae.html) route messages that were otherwise unroutable.
* [Sender-selected Distribution](https://www.rabbitmq.com/sender-selected.html) allows a publisher to decide where messages are routed directly.

#### Message Lifecycle

* [Per-Queue Message TTL](https://www.rabbitmq.com/ttl.html#per-queue-message-ttl) determines how long an unconsumed message can live in a queue before it is automatically deleted.
* [Per-Message TTL](https://www.rabbitmq.com/ttl.html#per-message-ttl) determines the TTL on a per-message basis.
* [Queue TTL](https://www.rabbitmq.com/ttl.html#queue-ttl) determines how long an unused queue can live before it is automatically deleted.
* [Dead Letter Exchanges](https://www.rabbitmq.com/dlx.html) ensure messages get re-routed when they are rejected or expire.
* [Queue Length Limit](https://www.rabbitmq.com/maxlength.html) allows the maximum length of a queue to be set.
* [Priority Queues](https://www.rabbitmq.com/priority.html) support the message priority field (in a slightly different way).

#### Authentication and Identity

* The [User-ID](https://www.rabbitmq.com/validated-user-id.html) message property is validated by the server.
* Clients that advertise the appropriate capability may receive explicit [authentication failure notifications](https://www.rabbitmq.com/auth-notification.html) from the broker.
* [update-secret](https://www.rabbitmq.com/amqp-0-9-1-reference.html#connection.update-secret) to be able to renew credentials for an active connection, when those credentials can expire.


### Client Documentation

* [Java Client API Guide](https://www.rabbitmq.com/api-guide.html)
  * `Client-provided connection name`
  * [`Channels and Concurrency Considerations (Thread Safety)`](https://www.rabbitmq.com/api-guide.html#concurrency)
  
    As a rule of thumb, `sharing Channel` instances between threads is something to be avoided. Applications should prefer 
    using a `Channel` per thread instead of sharing the same `Channel` across multiple threads.

    While some operations on channels are safe to invoke concurrently, some are not and will result in incorrect frame 
    interleaving on the wire, double acknowledgements and so on.

    Concurrent publishing on a shared channel can result in incorrect frame interleaving on the wire, triggering 
    a connection-level protocol exception and immediate connection closure by the broker. It therefore requires explicit 
    synchronization in application code (`Channel#basicPublish` must be invoked in a critical section). Sharing channels 
    between threads will also interfere with Publisher Confirms. Concurrent publishing on a shared channel is best avoided entirely, 
    e.g. by using a channel per thread.
  
    Consuming in one thread and publishing in another thread on a shared channel can be safe.

    When `manual acknowledgements` are used, it is important to consider what thread does the acknowledgement. 
    If it's different from the thread that received the delivery (e.g. `Consumer#handleDelivery` delegated delivery handling to a different thread), 
    acknowledging with the `multiple` parameter set to `true` is unsafe and will result in double-acknowledgements, 
    and therefore a channel-level protocol exception that closes the channel. Acknowledging a single message at a time can be safe.
  
  * [`Receiving Messages by Subscription ("Push API")`](https://www.rabbitmq.com/api-guide.html#consuming)

    Callbacks to `Consumers` are dispatched in a thread pool separate from the thread that instantiated its `Channel`. 
    This means that `Consumers` can safely call blocking methods on the `Connection` or `Channel`, such as `Channel#queueDeclare` or `Channel#basicCancel`.
  
    Each `Channel` has its own dispatch thread. For the most common use case of one `Consumer` per `Channel`, 
    this means `Consumers` do not hold up other `Consumers`. If you have multiple `Consumers` per `Channel` be aware that 
    `a long-running Consumer` may hold up dispatch of callbacks to other `Consumers` on that `Channel`.
  
  * [Handling unroutable messages](https://www.rabbitmq.com/api-guide.html#returning)
  * [Shutdown Protocol](https://www.rabbitmq.com/api-guide.html#shutdown)
    * Overview of the Client Shutdown Process
    * Information about the circumstances of a shutdown
    * Atomicity and use of the isOpen() method
  * [Consumer Operation Thread Pool](https://www.rabbitmq.com/api-guide.html#consumer-thread-pool)
  * [Service discovery with the AddressResolver interface](https://www.rabbitmq.com/api-guide.html#service-discovery-with-address-resolver)
  * [Heartbeat Timeout](https://www.rabbitmq.com/heartbeats.html)
  * [Support for Java non-blocking IO](https://www.rabbitmq.com/api-guide.html#java-nio)
  * [Automatic Recovery From Network Failures](https://www.rabbitmq.com/api-guide.html#recovery)
    * Connection Recovery
    * When Will Connection Recovery Be Triggered?
    * Recovery Listeners
    * Effects on Publishing
    * Failure Detection and Recovery Limitations
    * Manual Acknowledgements and Automatic Recovery
    * Channels Lifecycle and Topology Recovery
    * Unhandled Exceptions
  * [Metrics and Monitoring](https://www.rabbitmq.com/api-guide.html#metrics)

* [Spring AMQP](https://docs.spring.io/spring-amqp/docs/current/reference/html/)
  * [Annotation-driven Listener Endpoints](https://docs.spring.io/spring-amqp/docs/current/reference/html/#async-annotation-driven) -【重要】
  * [Exception Handling](https://docs.spring.io/spring-amqp/docs/current/reference/html/#exception-handling) -【重要】
  * [Recover/Requeue/Reject/Retry](https://docs.spring.io/spring-amqp/docs/current/reference/html/#resilience-recovering-from-errors-and-broker-failures) -【重要】
  * [Multiple Broker (or Cluster) Support](https://docs.spring.io/spring-amqp/docs/current/reference/html/#multi-rabbit) -【重要】
  * [Naming Connections](https://docs.spring.io/spring-amqp/docs/current/reference/html/#naming-connections)
  * [Blocked Connections and Resource Constraints](https://docs.spring.io/spring-amqp/docs/current/reference/html/#blocked-connections-and-resource-constraints)
  * [Using a Separate Connection](https://docs.spring.io/spring-amqp/docs/current/reference/html/#separate-connection)
  * [Connection and Channel Listeners](https://docs.spring.io/spring-amqp/docs/current/reference/html/#connection-channel-listeners)
  * [Logging Channel Close Events](https://docs.spring.io/spring-amqp/docs/current/reference/html/#channel-close-logging)
  * [Runtime Cache Properties](https://docs.spring.io/spring-amqp/docs/current/reference/html/#runtime-cache-properties)
  * [Adding Custom Client Connection Properties](https://docs.spring.io/spring-amqp/docs/current/reference/html/#custom-client-props)
  * [Publishing is Asynchronous — How to Detect Successes and Failures](https://docs.spring.io/spring-amqp/docs/current/reference/html/#publishing-is-async)
  * [Correlated Publisher Confirms and Returns](https://docs.spring.io/spring-amqp/docs/current/reference/html/#template-confirms)
  * [Strict Message Ordering in a Multi-Threaded Environment](https://docs.spring.io/spring-amqp/docs/current/reference/html/#multi-strict)
  * [Validated User Id](https://docs.spring.io/spring-amqp/docs/current/reference/html/#template-user-id)
  * [Message Builder API](https://docs.spring.io/spring-amqp/docs/current/reference/html/#message-builder)
  * [Consumer Events](https://docs.spring.io/spring-amqp/docs/current/reference/html/#consumer-events)
  * [Detecting Idle Asynchronous Consumers](https://docs.spring.io/spring-amqp/docs/current/reference/html/#idle-containers)
  * [Event Consumption](https://docs.spring.io/spring-amqp/docs/current/reference/html/#event-consumption)
  * [Broker Event Listener](https://docs.spring.io/spring-amqp/docs/current/reference/html/#broker-events)
  * [Logging Subsystem AMQP Appenders](https://docs.spring.io/spring-amqp/docs/current/reference/html/#logging) - 支持日志往AMQP发送
  * Transactions
    * [Transactions](https://docs.spring.io/spring-amqp/docs/current/reference/html/#transactions)
    * [Using transactions with RabbitMQ and SQL database in Spring Boot application](http://lifeinide.com/post/2017-12-29-spring-boot-rabbitmq-transactions/)

* [JMS Client](https://www.rabbitmq.com/jms-client.html)
  * [JMS Client Spec Compliance](https://www.rabbitmq.com/jms-client-compliance.html)

* [`Heartbeats` - Detecting Dead TCP Connections with Heartbeats and TCP Keepalives](https://www.rabbitmq.com/heartbeats.html) - 重要

* [AMQP 0-9-1 Reference](https://www.rabbitmq.com/amqp-0-9-1-reference.html)

* [AMQP 0-9-1 URI Spec](https://www.rabbitmq.com/uri-spec.html)
  * [URI Query Parameters](https://www.rabbitmq.com/uri-query-parameters.html)

* [Clients Libraries and Developer Tools](https://www.rabbitmq.com/devtools.html)

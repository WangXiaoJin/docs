# ZooKeeper安装及使用

## 1. 下载ZooKeeper

```bash
shell> wget https://archive.apache.org/dist/zookeeper/zookeeper-3.5.10/apache-zookeeper-3.5.10-bin.tar.gz
shell> tar xzf apache-zookeeper-3.5.10-bin.tar.gz
shell> mv apache-zookeeper-3.5.10-bin zookeeper
shell> cd zookeeper
```

## 2. 配置

需 Jdk8 或以上版本，配置`Java heap size`，避免内存设置过小导致频繁`swap`：

```bash
#创建java配置文件，保守建议：4GB内存设置为3GB，根据服务器使用情况而定。还可以设置其他的Java参数。
shell> echo 'export ZK_SERVER_HEAP="3072"' > conf/java.env
```

创建 `conf/zoo.cfg` 配置文件（可以拷贝`conf/zoo_sample.cfg`）

```bash
# 单位：毫秒 - 用于心跳检测。最小的session超时为 tickTime*2
tickTime=2000
# Amount of time, in ticks (see tickTime), to allow followers to connect and sync to a leader. 
# Increased this value as needed, if the amount of data managed by ZooKeeper is large.
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
#  (No Java system property) Limits the number of concurrent connections (at the socket level) that a single client, 
#  identified by IP address, may make to a single member of the ZooKeeper ensemble. 
#  This is used to prevent certain classes of DoS attacks, including file descriptor exhaustion. 
#  The default is 60. Setting this to 0 entirely removes the limit on concurrent connections.
maxClientCnxns=60
# Enable extended features such as the creation of TTL Nodes. They are disabled by default. 
# IMPORTANT: when enabled server IDs must be less than 255 due to internal limitations.
zookeeper.extendedTypesEnabled=true

# the minimum session timeout in milliseconds that the server will allow the client to negotiate. Defaults to 2 times the tickTime.
#minSessionTimeout=2

# the maximum session timeout in milliseconds that the server will allow the client to negotiate. Defaults to 20 times the tickTime.
#maxSessionTimeout=20

# Leader accepts client connections. Default value is "yes". The leader machine coordinates updates. 
# For higher update throughput at the slight expense of read throughput the leader can be configured to not accept clients and focus on coordination. 
# The default to this option is yes, which means that a leader will accept client connections.
#leaderServes=

# 集群配置
# 第一个端口号用于从节点连接主节点，第二个端口号用于leader选举
server.1=192.168.1.101:2888:3888
server.2=192.168.1.102:2888:3888
server.3=192.168.1.103:2888:3888

# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=8
# Purge task interval in hours. Set to "0" to disable auto purge feature
autopurge.purgeInterval=6

# A list of comma separated Four Letter Words commands that user wants to use. A valid Four Letter Words command must be put in this list else ZooKeeper server will not enable the command. 
# By default the whitelist only contains "srvr" command which zkServer.sh uses. The rest of four letter word commands are disabled by default. 
# Here's an example of the configuration that enables stat, ruok, conf, and isro command while disabling the rest of Four Letter Words command:
# If you really need enable all four letter word commands by default, you can use the asterisk option so you don't have to include every command one by one in the list. As an example：4lw.commands.whitelist=*
#4lw.commands.whitelist=stat, ruok, conf, isro

# Set to "false" to disable the AdminServer. By default the AdminServer is enabled.
#admin.enableServer=
# The address the embedded Jetty server listens on. Defaults to 0.0.0.0.
#admin.serverAddress=
# The port the embedded Jetty server listens on. Defaults to 8080.
#admin.serverPort=
# Set the maximum idle time in milliseconds that a connection can wait before sending or receiving data. Defaults to 30000 ms.
#admin.idleTimeout=
# The URL for listing and issuing commands relative to the root URL. Defaults to "/commands".
#admin.commandURL=
```

> 参考文档：<https://zookeeper.apache.org/doc/r3.5.10/zookeeperAdmin.html>

### 初始化数据目录及myid

每个节点都需初始化数据目录及myid文件，myid 值和集群配置中的配置保持一致。

```
shell> ./zkServer-initialize.sh --myid=1
```

### 服务及端口配置

A client port of a server is the port on which the server accepts client connection requests. Starting with 3.5.0 the clientPort and clientPortAddress configuration parameters should no longer be used. 
Instead, this information is now part of the server keyword specification, which becomes as follows:

```
server.<positive id> = <address1>:<port1>:<port2>[:role];[<client port address>:]<client port>**
```

The client port specification is to the right of the semicolon. The client port address is optional, and if not specified it defaults to "0.0.0.0". 
As usual, role is also optional, it can be participant or observer (participant by default).

Examples of legal server statements:

```
server.5 = 125.23.63.23:1234:1235;1236
server.5 = 125.23.63.23:1234:1235:participant;1236
server.5 = 125.23.63.23:1234:1235:observer;1236
server.5 = 125.23.63.23:1234:1235;125.23.63.24:1236
server.5 = 125.23.63.23:1234:1235:participant;125.23.63.23:1236
```

## 3. 启动服务

```bash
shell> bin/zkServer.sh start
```

## 4. 连接

```bash
shell> bin/zkCli.sh -server 127.0.0.1:2181
```

> znode存储的数据应小于1M,client/server都是做此检查的。  
  client连接`127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002/app/a`来改变根路径  
  client连接zookeeper server时可以传一个session timeout参数，这个参数值必须在大于等于`tickTime * 2`，小于等于 `tickTime * 20`  

## 5. Zookeeper GUI

* [PrettyZoo](https://github.com/vran-dev/PrettyZoo)

## 6. Supervision

是否需要此功能依据个人情况而定。

* [daemontools](http://cr.yp.to/daemontools.html)
* [SMF](http://en.wikipedia.org/wiki/Service_Management_Facility)
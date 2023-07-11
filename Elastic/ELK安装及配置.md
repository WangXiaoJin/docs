# 安装ELK组件

## Elasticsearch

### 前置条件

#### 1. Virtual memory

```shell
# 永久生效
shell> vim /etc/sysctl.conf
vm.max_map_count = 262144

# 临时生效
shell> sysctl -w vm.max_map_count=262144
```

> 以`RPM`或`Debian`包运行的ES默认已经配置

#### 2. Max files descriptors

确保运行ES的用户`max files descriptors`值最低为 65536。

> 以`RPM`或`Debian`包运行的ES默认已经配置

参考链接:
* [Configuring system settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html)  
* [File Descriptors](https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html)

#### 3. Number of threads

确保运行ES的用户`nproc`值最低为 4096。

```shell
# 永久生效
shell> vim /etc/security/limits.conf
elasticsearch   -   nproc    10240

# 临时生效
shell> ulimit -u 10240 
```

> 以`RPM`或`Debian`包运行的ES默认已经配置

> [参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/max-number-of-threads.html)

#### 4. Disable swapping

```shell
# 临时禁用
# 无需重启 Elasticsearch
shell> sudo swapoff -a

# 永久禁用
# 注释掉包含`swap`关键字的行
shell> vim /etc/fstab
```

> [参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html)

#### 5. TCP retransmission timeout

`net.ipv4.tcp_retries2`默认配置为15，按公式计算后超时时间为900秒。

```shell
# 临时配置
shell> sysctl -w net.ipv4.tcp_retries2=5

# 永久禁用
shell> vim /etc/sysctl.conf
net.ipv4.tcp_retries2 = 5
```

> 注：如果网络环境不稳定可以适当往上调整

> [参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config-tcpretries.html)

#### 6. Heap size settings

By default, Elasticsearch automatically sets the JVM heap size based on a node’s [roles](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-roles) 
and total memory. We recommend the default sizing for most production environments.

如想自定义 JVM 配置，最好是确保heap size低于`zero-based compressed oops`的阀值，26GB适用于大多数系统，某些系统能达到30GB，你可以在启动
ES时增加以下JVM参数来验证：`-XX:+UnlockDiagnosticVMOptions -XX:+PrintCompressedOopsMode`

```
heap address: 0x000000011be00000, size: 27648 MB, zero based Compressed Oops
```

#### 7. Temporary directory settings

By default, Elasticsearch uses a private temporary directory that the startup script creates immediately below the system temporary directory.

On some Linux distributions, a system utility will clean files and directories from `/tmp` if they have not been 
recently accessed. This behavior can lead to the private temporary directory being removed while Elasticsearch is running 
if features that require the temporary directory are not used for a long time. Removing the private temporary directory 
causes problems if a feature that requires this directory is subsequently used.

If you install Elasticsearch using the `.deb` or `.rpm` packages and run it under `systemd`, 
the private temporary directory that Elasticsearch uses is excluded from periodic cleanup.

If you intend to run the `.tar.gz` distribution on Linux or MacOS for an extended period, consider creating 
a dedicated temporary directory for Elasticsearch that is not under a path that will have old files and directories cleaned from it. 
This directory should have permissions set so that only the user that Elasticsearch runs as can access it. 
Then, set the `$ES_TMPDIR` environment variable to point to this directory before starting Elasticsearch.

```bash
shell> wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.2-linux-x86_64.tar.gz
shell> tar -xzf elasticsearch-8.6.2-linux-x86_64.tar.gz
shell> cd elasticsearch-8.6.2/
shell> groupadd elasticsearch   #添加elasticsearch用户组
shell> useradd elasticsearch -g elasticsearch -p espassword #添加elasticsearch用户
shell> mkdir -p /var/log/elasticsearch /var/data/elasticsearch #用于存放ES的日志和数据
shell> chown -R elasticsearch:elasticsearch elasticsearch-5.5.0 /var/log/elasticsearch /var/data/elasticsearch
shell> su elasticsearch #切换到elasticsearch用户
# -d 以守护进程运行，-p pid 把进程id记录到pid文件中
shell> ./bin/elasticsearch -d -p pid
shell> kill `cat pid` #关闭elasticsearch
```




 
修改`elasticsearch.yml`配置文件，这里我举例了集群配置，一共有`192.168.206.130`、`192.168.206.134`、`192.168.206.135`三个节点：
 
Node1配置：
```yaml
cluster.name: app-log-prod
#node.name: ${HOSTNAME} 可以设置系统环境变量
node.name: app-log-data-1
path.data: /var/data/elasticsearch
path.logs: /var/log/elasticsearch
#plugins: /path/to/plugins
network.host: 192.168.206.130
http.port: 9200
transport.tcp.port: 9300
# 通过文件动态配置 seed nodes
discovery.seed_providers: file
# Provides a list of the addresses of the master-eligible nodes in the cluster
discovery.seed_hosts:
  - 192.168.206.130:9300
  - 192.168.206.131:9300
  - 192.168.206.132:9300
# 如果在启动ES时抛出“unable to install syscall filter”警告，则禁用system_call_filter。
# 因为某些系统（CentOS6）不支持seccomp，而ES默认开启system_call_filter，所以会抛出警告
#bootstrap.system_call_filter: false
#---------- 集群配置 ---------------
#当填写域名时循环处理多个IP。端口号依次取值1.transport.profiles.default.port 2.transport.tcp.port
discovery.zen.ping.unicast.hosts: ["192.168.206.130", "192.168.206.134", "192.168.206.135"]
#不配置此参数时，默认为1。计算公式：(master_eligible_nodes / 2) + 1，master_eligible_nodes为master候选节点数
discovery.zen.minimum_master_nodes: 2
#---------- 集群配置 ---------------
```

Node2配置（Node3的配置参考Node2修改下）：
```yaml
cluster.name: app-log-prod
#node.name: ${HOSTNAME} 可以设置系统环境变量
node.name: app-log-data-2
path.data: /var/data/elasticsearch
path.logs: /var/log/elasticsearch
#plugins: /path/to/plugins
network.host: 192.168.206.134
http.port: 9200
transport.tcp.port: 9300
# 如果在启动ES时抛出“unable to install syscall filter”警告，则禁用system_call_filter。
# 因为某些系统（CentOS6）不支持seccomp，而ES默认开启system_call_filter，所以会抛出警告
#bootstrap.system_call_filter: false
#---------- 集群配置 ---------------
#当填写域名时循环处理多个IP。端口号依次取值1.transport.profiles.default.port 2.transport.tcp.port
discovery.zen.ping.unicast.hosts: ["192.168.206.130", "192.168.206.134", "192.168.206.135"]
#不配置此参数时，默认为1。计算公式：(master_eligible_nodes / 2) + 1，master_eligible_nodes为master候选节点数
discovery.zen.minimum_master_nodes: 2
#---------- 集群配置 ---------------
```
 
**防火墙配置：**
```bash
shell> vim /etc/sysconfig/iptables
# 端口号根据自己的配置而定，增加下面配置
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9300 -j ACCEPT
shell> service iptables restart
```

通过命令行修改cluster/node name：  
`./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name`  
通常集群配置项(如：`cluster.name`)应该配置在`elasticsearch.yml`文件中，特定节点配置项（如：`node.name`）应该通过命令行参数配置。

> ES默认会加载$ES_HOME/config/elasticsearch.yml这个配置文件，此配置文件中任意配置项都可以通过命令行参数`-Ekey=value`
格式配置。例：`./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1`。  
配置文件目录可以通过`-Epath.conf=/path/to/my/config/`来修改。

> Elasticsearch 默认配置运行 64-bit JVMs。如果你运行了32-bit JVM，你必须删除`jvm.options`中`-server`配置。
且重新配置`thread stack size`值，修改`-Xss1m`为`-Xss320k`。
 
**节点配置：**
 
* `discovery.zen.master_election.ignore_non_master_pings` - 默认值：false。设置为true时，在选举master节点时，
会忽略掉非候选master节点（`node.master: false`）的pings
* `discovery.zen.minimum_master_nodes` - 默认值：1。当选举Master节点时，必须最少有多少个Master候选节点处于活动状态，否则不能选举Master节点。
此配置项同时意味着当前Cluster至少有多少个Master候选节点有效，如果不满足此条件则当前Master节点降级，同时试图选出新的Master节点。  
  如果是集群环境，则Master候选节点数最好大于等于3，且`minimum_master_nodes`配置为：`(master_eligible_nodes / 2) + 1`
 
    minimum_master_nodes可以通过cluster API动态设值：  
    ```
    PUT _cluster/settings
    {
      "transient": {
        "discovery.zen.minimum_master_nodes": 2
      }
    }
    ```
* 集群环境下，建议配置专用节点。不同的节点充当不同的角色，执行不同的功能。Master候选节点数: 3，minimum_master_nodes: 2，
后续新增的节点设值为data或ingest节点。配置如下：
    ```
    node.master: true 	#(default: true - Master候选节点)
    node.data: false	#(default: true - 数据节点)
    node.ingest: false	#(default: true - 写数据之前对其进行处理)
    ```


### ES 参考文档

#### Set Up

* [Discovery and cluster formation settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-settings.html)
* [Cluster-level shard allocation and routing settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html)
* [Miscellaneous cluster settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html)
  * `cluster.blocks.read_only`
  * `cluster.blocks.read_only_allow_delete`
  * `cluster.max_shards_per_node`
  * `cluster.max_shards_per_node.frozen`
  * `User-defined cluster metadata` - Any information stored in user-defined cluster metadata will be viewable by anyone with access to the [Cluster Get Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-get-settings.html) API, and is recorded in the Elasticsearch logs.
  * `cluster.indices.tombstones.size`
  * `Logger`
  * `cluster.persistent_tasks.allocation.enable`
  * `cluster.persistent_tasks.allocation.recheck_interval`
* [Field data cache settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-fielddata.html)
* [Index management settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-management-settings.html)
* [Index recovery settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/recovery.html)
* [Local gateway settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html)
* [Logging](https://www.elastic.co/guide/en/elasticsearch/reference/current/logging.html)
* [Node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
  * Node roles
  * Coordinating node
  * Node data path
* [Networking](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)
  * Network settings
  * Request tracing
  * Networking threading model
* [Query cache index settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-cache.html)
* [Security settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html)
* [Shard request cache settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-request-cache.html)
* [Snapshot and restore settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-settings.html)
* [Thread pools](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)
* [Watcher settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/notification-settings.html)
* [Advanced configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html) - 使用默认值，不建议调整
  * JVM options



* [Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)
* [Important Elasticsearch configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)
* [Node详解及配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html) - 重要
 
> 注：ES的索引名(_index)必须为小写字母，不能以下划线开头，且不能包含逗号。  
> 类型名(_type)可以为大、小写字母，不能以下划线或句点开头，且不能包含逗号，长度限制在256个字符之内

## Kibana

```bash
shell> wget https://artifacts.elastic.co/downloads/kibana/kibana-5.5.0-linux-x86_64.tar.gz
shell> sha1sum kibana-5.5.0-linux-x86_64.tar.gz
shell> tar -xzf kibana-5.5.0-linux-x86_64.tar.gz
shell> mkdir -p /data/logs/kibana  #创建Kibana日志目录
## chown -R kibana:kibana /data/logs/kibana kibana-5.5.0-linux-x86_64 #当你用kibana用户起动Kibana时需要此步骤，否则不需要
shell> cd kibana-5.5.0-linux-x86_64
shell> nohup ./bin/kibana &
```
 
* 修改`kibana.yml`配置：
 
    ```yaml
    server.port: 5601
    server.host: "192.168.206.136"
    elasticsearch.url: "http://192.168.206.136:9200"
    pid.file: /var/run/kibana.pid
    logging.dest: /data/logs/kibana/kibana.log
    #i18n.defaultLocale: "en"
    ```
 
* 防火墙配置：
     
    ```bash
    shell> vim /etc/sysconfig/iptables
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 5601 -j ACCEPT
    shell> service iptables restart
    ```
 
* 负载均衡配置 - 因Kibana不支持直接连接配置多个ES地址，所以当前最优方案是Kibana于ES集群分开部署，在当前Kibana机器部署一个ES协调节点，此ES Node负责分发Kibana请求，
不参与ES集群的数据存储等相关操作。配置参考如下：
    ```yaml
    cluster.name: app-log-prod
    node.name: app-log-kibana
    path.data: /var/data/elasticsearch
    path.logs: /var/log/elasticsearch
    network.host: 192.168.206.136
    http.port: 9200
    transport.tcp.port: 9300
    # 如果在启动ES时抛出“unable to install syscall filter”警告，则禁用system_call_filter。
    # 因为某些系统（CentOS6）不支持seccomp，而ES默认开启system_call_filter，所以会抛出警告
    #bootstrap.system_call_filter: false
    #当填写域名时循环处理多个IP。端口号依次取值1.transport.profiles.default.port 2.transport.tcp.port
    discovery.zen.ping.unicast.hosts: ["192.168.206.130", "192.168.206.134", "192.168.206.135"]
    #不配置此参数时，默认为1。计算公式：(master_eligible_nodes / 2) + 1，master_eligible_nodes为master候选节点数
    discovery.zen.minimum_master_nodes: 2
    node.master: false
    node.data: false
    node.ingest: false
    ```
 
* 访问`http://192.168.206.136:5601/status` ，显示服务状态、资源使用情况及已安装插件

* 第一次访问Kibana需要配置默认`index pattern`
     
  * 配置`Index name or pattern`值为`app-*`（依据存入ES索引名而定）
  * `Time Filter field name`选择`@timestamp`
  * 点击`Create`按钮
 
* Kibana高级配置：
  * `Management` > `Advanced Settings` > `dateFormat` : `YYYY-MM-DD HH:mm:ss.SSS`
  * 格式化字段 - [文档](https://www.elastic.co/guide/en/kibana/current/field-formatters-string.html)
  * 脚本字段 - [文档](https://www.elastic.co/guide/en/kibana/current/scripted-fields.html)

* 推荐插件
  * [LogTrail](https://github.com/sivasamyk/logtrail) - 实时查看、分析日志，类似于tail命令。灵感来自于[Papertrail](https://papertrailapp.com/)
  * [Own Home](https://github.com/wtakase/kibana-own-home) - 提供登录、多租户功能
  * [X-Pack](https://www.elastic.co/cn/products/x-pack) - 官方插件，收费插件。提供登录、角色、权限、实时监控、表报生成、机器学习

## Logstash

Logstash-5.4依赖Java 8，不支持Java 9。

```bash
shell> wget https://artifacts.elastic.co/downloads/logstash/logstash-5.5.0.tar.gz
shell> tar -xvf logstash-5.5.0.tar.gz
shell> cd logstash-5.5.0
shell> bin/logstash-plugin update logstash-filter-dissect #更新dissect插件，5.5.0版本需要自己更新
shell> bin/logstash-plugin update logstash-input-dead_letter_queue #更新dead_letter_queue插件，5.5.0版本需要自己更新
shell> nohup bin/logstash -f logstash.conf &   #执行之前修改下logstash.yml、logstash.conf配置文件
```
 
**修改`logstash.yml`配置，[参考文档](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html)：**

```yaml
#node.name: test #logstash实例名，默认值为机器的hostname，如果你想在一台机子上部署多个logstash实例，请自定义此配置

path.data: /data/logstash
#config.reload.automatic: true #定期自动加载改动的配置文件，不需要手动重启logstash

#queue是否存储到磁盘，默认只存储到缓存中，`persisted`可以防止logstash不正常关闭时丢失数据。
#根据使用场景而定，重要的数据或者需要`large buffers`可以使用`persisted`
#当开启persisted后，可以不使用消息中间件（如：Redis, RabbitMQ, Apache Kafka）
queue.type: persisted
queue.max_bytes: 1gb  #默认值1gb，超过1gb后input端将会被阻塞，将服务压力留在input端
dead_letter_queue.enable: true # 5.5.0支持的特性，把写入ES失败的Event写入Dead Letter Queue，当前版本只支持ES
path.logs: /data/logs/logstash
#配置slowlog
#slowlog.threshold.warn: 2s
#slowlog.threshold.info: 1s
#slowlog.threshold.debug: 500ms
#slowlog.threshold.trace: 100ms
```
 
启用persistent queues优势，参考文档 [persistent-queues](https://www.elastic.co/guide/en/logstash/current/persistent-queues.html)：

* 爆发性流量不需要额外的缓冲机制，如Redis, RabbitMQ, Apache Kafka等。logstash可以将所有的events写入磁盘中。
* 确保数据不会丢失。不是100%不丢失，根据`queue.checkpoint.writes`配置而定。
 
**新建`es-template-app.json`文件：**
```json
{
  "template" : "app-*",
  "version" : 50001,
  "settings" : {
    "index.refresh_interval" : "5s"
  },
  "mappings" : {
    "_default_" : {
      "_all" : {"enabled" : true, "norms" : false}, 
      "dynamic_templates" : [ {
        "message_field" : {
          "path_match" : "message",
          "match_mapping_type" : "string",
          "mapping" : {
            "type" : "text",
            "norms" : false
          }
        }
      }, {
        "string_fields" : {
          "match" : "*",
          "match_mapping_type" : "string",
          "mapping" : {
            "type" : "text", "norms" : false,
            "fields" : {
              "keyword" : { "type": "keyword", "ignore_above": 256 }
            }
          }
        }
      } ],
      "properties" : {
        "@timestamp": { "type": "date", "include_in_all": false },
        "@version": { "type": "keyword", "include_in_all": false },
        "msg": { "type": "text" },
        "app": { "type": "keyword" },
        "offset": { "type": "long" },
        "pid": { "type": "integer" },
        "type": { "type": "keyword", "include_in_all": false },
        "env": { "type": "keyword" },
        "threadName": { "type": "keyword" },
        "logLevel": { "type": "keyword" },
        "logHost": { "type": "keyword" },
        "host": { "type": "keyword" },
        "beat": {
          "properties": {
            "hostname": { "type": "keyword" },
            "name": { "type": "keyword" },
            "input_type": { "type": "keyword" },
            "version": { "type": "keyword" }
          }
        },
        "geoip"  : {
          "dynamic": true,
          "properties" : {
            "ip": { "type": "ip" },
            "location" : { "type" : "geo_point" },
            "latitude" : { "type" : "half_float" },
            "longitude" : { "type" : "half_float" }
          }
        }
      }
    }
  }
}
```
 
**新建`es-template-dlq.json`文件（`dead_letter_queue`）：**
```json
{
  "template" : "dlq-*",
  "version" : 50001,
  "settings" : {
    "index.refresh_interval" : "5s"
  },
  "mappings" : {
    "_default_" : {
      "_all" : {"enabled" : true, "norms" : false}, 
      "dynamic_templates" : [ {
        "message_field" : {
          "path_match" : "message",
          "match_mapping_type" : "string",
          "mapping" : {
            "type" : "text",
            "norms" : false
          }
        }
      }, {
        "string_fields" : {
          "match" : "*",
          "match_mapping_type" : "string",
          "mapping" : {
            "type" : "text", "norms" : false,
            "fields" : {
              "keyword" : { "type": "keyword", "ignore_above": 256 }
            }
          }
        }
      } ],
      "properties" : {
        "@timestamp": { "type": "date", "include_in_all": false },
        "@version": { "type": "keyword", "include_in_all": false },
        "type": { "type": "keyword", "include_in_all": false },
        "host": { "type": "keyword" },
        "dead_letter_queue": {
          "properties": {
            "plugin_id": { "type": "keyword" },
            "reason": { "type": "text" },
            "entry_time": { "type": "date" },
            "plugin_type": { "type": "keyword" }
          }
        }
      }
    }
  }
}
```
 
**手动加载ES Template：**
```bash
shell> curl -H 'Content-Type: application/json' -XPUT 'http://192.168.206.130:9200/_template/app' -d@es-template-app.json
shell> curl -H 'Content-Type: application/json' -XPUT 'http://192.168.206.130:9200/_template/dlq' -d@es-template-dlq.json
```

**配置`logstash.conf`文件，文本必须使用UTF-8格式：**
```
input {
  beats {
    port => 5044
  }
  
  # 5.5.0支持的特性
  # 开启commit_offsets时（默认开启），只有在Logstash关闭时才会把记录的状态写入sincedb中。
  # 当出现非正常关闭Logstash（宕机、kill -9等），下次启动Logstash会出现相同Event重复处理的现象。
  # 希望官方以后能对此插件优化，周期性更新状态到sincedb中，而不是等到关闭Logstash时才更新。
  dead_letter_queue {
    path => "/data/logstash/dead_letter_queue"
  }
}

filter {
  if [@metadata][dead_letter_queue] {
    # 处理dead_letter_queue Event
    mutate {
      copy => { "[@metadata][dead_letter_queue]" => "dead_letter_queue" }
      add_field => {
        "[@metadata][index]" => "dlq"
      }
    }
  } else {
    if [@metadata][beat] == "filebeat" {
      # 当前版本[type]属性值和[@metadata][type]由FileBeat自动赋值
      mutate {
        add_field => { "[beat][source]" => "%{source}" }
      }
      mutate {
        rename => { "input_type" => "[beat][input_type]" }
        # source路径格式 /data/logs/app/{env}/{project}/{log-name}.log
        split => { "source" => "/" }
        add_field => {
          "env" => "%{source[4]}"
          "app" => "%{source[5]}"
          "[@metadata][index]" => "app-%{env}"
        }
        remove_field => [ "source" ]
      }
      mutate {
        replace => { "type" => "%{app}" }
      }
    }
    
    if [app] {
      # 这里使用dissect而不用grok的原因是，grok用正则来匹配，比较耗性能，grok适用于复杂格式
      # dissect表达式中间的空格字符可以匹配多个空格，grok想匹配多个空格必须通过正则来实现
      # 解析格式：2017-07-10 18:59:49.147 INFO [{Hostname}] [{PID}] [{ThreadName}] c.e.u.c.SpringMvcConfig : xxxxx
      dissect {
        mapping => {
          "message" => "%{ts} %{+ts} %{logLevel} [%{logHost}] [%{pid}] [%{threadName}] %{msg}"
        }
        convert_datatype => {
          "pid" => "int"
        }
      }
    }
  
    # [ts]替换掉[@timestamp]值
    if [ts] {
      date {
        match => [ "ts", "ISO8601" ]
        remove_field => [ "ts"]
      }
    }
  }
}

output {
  # metadata:true，打印metadata信息，调试使用
  #stdout { codec => rubydebug { metadata => true } }
  
  elasticsearch {
    hosts => ["192.168.206.130:9200", "192.168.206.134:9200", "192.168.206.135:9200"]
    http_compression => true
    # sniffing会返回所有开启http.enabled的cluster nodes（包括Master Node），
    # 如果你启用了Master Node，则关闭此Master Node的http.enabled
    sniffing => true
    manage_template => false
    index => "%{[@metadata][index]}-%{+YYYY.MM.dd}" 
    document_type => "%{type}" 
  }
}
```
 
**防火墙配置：**
```bash
shell> vim /etc/sysconfig/iptables
# 端口号根据自己的配置而定，增加下面配置
-A INPUT -m state --state NEW -m tcp -p tcp --dport 5044 -j ACCEPT
shell> service iptables restart
```
> 注：如果想Logstash直接解析JSON格式数据，请参考案例[JSON Logback with Logstash](https://cloud.spring.io/spring-cloud-static/Finchley.SR2/single/spring-cloud.html#_json_logback_with_logstash)

> `bin/logstash`命令[参考文档](https://www.elastic.co/guide/en/logstash/current/running-logstash-command-line.html)，或者通过`bin/logstash --help`查看帮助信息。  

> [`Config File`语法](https://www.elastic.co/guide/en/logstash/current/configuration.html)，包含此链接下所有的二级目录。

> [plugins文档](https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html)

## Beats
    
Beats不依赖于JDK，相对于Logstash来说，Beats更轻量级且消耗更少的系统资源。

```bash
shell> curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.1-linux-x86_64.tar.gz
shell> tar xzvf filebeat-8.6.1-linux-x86_64.tar.gz
shell> cd filebeat-8.6.1-linux-x86_64
shell> nohup ./filebeat &   #执行之前修改下filebeat.yml配置文件，参考filebeat.full.yml
```
 
`filebeat.yml`配置示例：
```yaml
#name: #设置beat.name值，默认值：hostname
path.data: /data/filebeat
path.logs: /data/logs/filebeat

filebeat.prospectors:
- input_type: log
  enabled: false
  paths:
    - /var/log/*
  
- input_type: log
  paths:
    - /data/logs/app/default/*/*.log
    - /data/logs/app/dev/*/*.log
    - /data/logs/app/test/*/*.log
    - /data/logs/app/pre/*/*.log
    - /data/logs/app/prod/*/*.log
  # 6.0版本已废弃此配置，可使用fields代替
  document_type: app
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

output.logstash:
  enabled: true
  hosts: ["192.168.206.134:5044", "192.168.206.135:5044"]
  loadbalance: true
  worker: 2  # Number of workers per Logstash host

#输出到控制台 - 测试使用
output.console:
  enabled: false
  pretty: true

filebeat.shutdown_timeout: 5s

logging.level: info
logging.to_files: true
logging.to_syslog: false
logging.files:
  name: filebeat
  keepfiles: 7
```
 
配置文件语法注意事项：
* `inputs.type: log`
  * `close_removed` - 默认启用，当文件删除时关闭文件handler，判断的依据为`file name`，而不是`inode`。
  * `close_renamed` - 默认值：false，当文件 renamed 时，关闭文件handler，当日志文件滚动生成且保留最近N个文件时启用此参数非常有效，
  可以关闭 处于deleted状态 的文件handler，避免日志生成速度大于消费速度而导致磁盘爆满的问题。如果文件renamed后，新的文件名匹配监控的
  path，则日志不会丢失，如果renamed后不匹配path，则后续日志不会采集。
  * `close_timeout` - 日志文件只采集指定时间段，采集超出时则关闭 file handler，当下扫描后发现文件没采集完则继续采集，采集剩余时间会重置。
  此配置可能会把多行日志分成两次发送，从而导致日志不全。
* Filebeat使用的`glob pattern`相对于[Logstash glob pattern](https://www.elastic.co/guide/en/logstash/current/glob-support.html)来说简单了很多，
* 字符窜有三种格式，双引号、单引号、无引号。
    * 双引号 - 支持转义字符‘\’，转义特殊字符
    * 单引号 - 不支持字符转义，内部有单引号时，需在单引号前面再增加一个单引号：`'This is a xx''xx.'`
    * 无引号 - 不支持字符转义，且不能出现在YAML中有特殊意义的字符
     
     >正则表达式强烈建议使用单引号格式，因为正则表达式和YAML都是通‘\’来转义字符，当用单引号格式时，
    ‘\’不会被YAML解析成转义字符，只有在双引号格式下才会被YAML解析成转义字符。
* 时间格式，有效的时间单位：`ns`, `us`, `ms`, `s`, `m`, `h`
 
    ```yaml
    duration1: 2.5s
    duration2: 6h
    duration_disabled: -1s
    ```
* Format String (sprintf)，格式：`%{<accessor>:default value}`
     
    ```yaml
    constant-format-string: 'constant string'
    field-format-string: '%{[fieldname]} string'
    format-string-with-date: '%{[fieldname]}-%{+yyyy.MM.dd}'

* 避免在数值前面加`0`，因为YAML会解析成八进制。可以用单引号包裹或去除前缀。
 
* 配置文件中使用环境变量 - [文档](https://www.elastic.co/guide/en/beats/filebeat/current/using-environ-vars.html)


### 参考文档

* [ELK各版本及系统支持矩阵](https://www.elastic.co/cn/support/matrix)
* [Filebeat各环境安装**目录**](https://www.elastic.co/guide/en/beats/filebeat/current/directory-layout.html)
* [Filebeat Command 参考手册](https://www.elastic.co/guide/en/beats/filebeat/current/command-line-options.html)
  * `export` - Exports the configuration, index template, ILM policy, or a dashboard to stdout.
  * `help` - Shows help for any command.
  * `keystore` - Manages the secrets keystore.
  * `modules` - Manages configured modules.
  * `run` - Runs Filebeat. This command is used by default if you start Filebeat without specifying a command.
  * `setup` - Sets up the initial environment, including the index template, ILM policy and write alias, Kibana dashboards (when available), and machine learning jobs (when available).
  * `test` - Tests the configuration.
  * `version` - Shows information about the current version.
* [Beat 配置文件语法](https://www.elastic.co/guide/en/beats/libbeat/current/config-file-format.html) - **非常重要**，同样适用于ELK
  * [Config file data types](https://www.elastic.co/guide/en/beats/libbeat/8.4/config-file-format-type.html)
  * [Environment variables](https://www.elastic.co/guide/en/beats/libbeat/8.4/config-file-format-env-vars.html)
  * [Reference variables](https://www.elastic.co/guide/en/beats/libbeat/8.4/config-gile-format-refs.html)
  * [Config file ownership and permissions](https://www.elastic.co/guide/en/beats/libbeat/8.4/config-file-permissions.html)
  * [YAML tips and gotchas](https://www.elastic.co/guide/en/beats/libbeat/8.4/config-file-format-tips.html)
* [FileBeat调试文档](https://www.elastic.co/guide/en/beats/filebeat/current/enable-filebeat-debugging.html)


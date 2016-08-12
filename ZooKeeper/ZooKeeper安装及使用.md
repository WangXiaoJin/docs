## ZooKeeper安装及使用

1. 下载ZooKeeper

	```bash
	wget http://mirrors.cnnic.cn/apache/zookeeper/stable/zookeeper-3.4.8.tar.gz
	tar xzf zookeeper-3.4.8.tar.gz
	mv zookeeper-3.4.8 zookeeper
	cd zookeeper
	```

2. 配置

	配置`Java heap size`，避免内存设置过小导致频繁`swap`：
	
	```bash
	#创建java配置文件，保守建议：4GB内存设置为3GB，根据服务器使用情况而定。还可以设置其他的Java参数。
	echo 'export JVMFLAGS="-Xmx3072m"' > conf/java.env
	```
	
	**`conf/zoo.cfg`（安装目录下有`conf/zoo_sample.cfg`）,具体请查阅<https://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_configuration>：**
	
	```bash
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
	maxClientCnxns=60
	
	# 集群配置
	server.1=192.168.1.101:2888:3888
	server.2=192.168.1.102:2888:3888
	server.3=192.168.1.103:2888:3888
	
	# The number of snapshots to retain in dataDir
	#autopurge.snapRetainCount=3
	# Purge task interval in hours. Set to "0" to disable auto purge feature
	#autopurge.purgeInterval=1
	```

	> 注： 集群配置项中第一个port为zookeeper各服务间日常通信使用，第二个port用于选举leader时使用。  
	>	集群配置时，需要在`dataDir`目录里创建一个`myid`文件，文件里只需存储`server.1`配置项对应的数值`1`，这个值作为机器ID使用。
		此值取值范围 `1 - 255`每个`zookeeper server`都需要配置。  
	> `3.5.0`版本增加了许多新功能，参考官网地址：<https://zookeeper.apache.org/doc/trunk/zookeeperReconfig.html>
	
	修改防火墙：
	
	```bash
	shell> vim /etc/sysconfig/iptables
	# 端口号根据自己的配置而定
	shell> # 加入：-A INPUT -m state --state NEW -m tcp -p tcp --dport 2181 -j ACCEPT
	shell> # 加入：-A INPUT -m state --state NEW -m tcp -p tcp --dport 2888 -j ACCEPT
	shell> # 加入：-A INPUT -m state --state NEW -m tcp -p tcp --dport 3888 -j ACCEPT
	shell> service iptables restart
	```

3. 启动服务

	```bash
	bin/zkServer.sh start
	```

4. 连接

	```bash
	bin/zkCli.sh -server 127.0.0.1:2181
	```

	> znode存储的数据应小于1M,client/server都是做此检查的。  
	  client连接`127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002/app/a`来改变根路径  
      client连接zookeeper server时可以传一个session timeout参数，这个参数值必须在大于等于`tickTime * 2`，小于等于 `tickTime * 20`  


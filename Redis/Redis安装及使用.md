## Redis安装及使用

### Redis安装

```bash
wget http://download.redis.io/releases/redis-3.0.7.tar.gz
tar xzf redis-3.0.7.tar.gz
cd redis-3.0.7
make
#可选操作。把Redis二进制可执行文件塞入/usr/local/bin,redis_init_script中默认可执行文件路径就为/usr/local/bin。make PREFIX=xxx install参数可重新指定bin文件安装目录
make install
#redis_init_script中的端口号默认为6379，如不是6379，请cp之后修改redis_init_script中的REDISPORT参数
cp utils/redis_init_script /etc/init.d/redis_6379
#此目录下存放配置文件
mkdir /etc/redis
#Redis存放RDB、AOF的目录
mkdir -p /var/redis/6379
cp redis.conf /etc/redis/6379.conf
```

### Redis服务配置提示

* 建议Redis服务部署在Linux系统中

* 官方推荐Linux内核中设置**overcommit memory setting to 1**，不然启动Redis服务会提示警告信息。设置为1的情况下，不管内存使用情况如何，不会返回申请内存失败提示信息。当运行时发现内存不足时，会触发OOM killer(OOM=out-of-memory)**杀死部分进程**以释放出足够的内存。看到这是不是有种吓尿的感觉（kill其他线程。会不会干掉有用的线程？）。我觉得在没有完全了解OOM killer原理的情况下，这个参数慎用啊，但是官方又推荐使用，官方也没有做出更多的解释以撤销我的顾虑。只能等以后时间再深入研究OOM killer了。参考链接：<http://www.hulkdev.com/posts/about_overcommit_memory>、<http://jangzq.info/2015/06/27/overcommit_memory/>、<http://mdba.cn/?p=627>、<http://unix.stackexchange.com/questions/153585/how-oom-killer-decides-which-process-to-kill-first>。修改方式：
	* 编辑`/etc/sysctl.conf`，改`vm.overcommit_memory=1`，然后`sysctl -p`使配置文件生效
	* `sysctl vm.overcommit_memory=1`，重启会失效
	* `echo 1 > /proc/sys/vm/overcommit_memory`， 重启会失效

* 确保关闭linux内核特性：`transparent huge pages`，不然启动Redis服务会提示警告信息。请参考[关闭TransparentHugePages (THP)](..linux/关闭TransparentHugePages (THP).md)、<http://blog.csdn.net/chosen0ne/article/details/46625359>

* `somaxconn`定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为128。而Redis的配置为511，所以启动Redis服务时会有警告提示。执行如下命令就OK了：
	
	> echo 511 > /proc/sys/net/core/somaxconn
	> echo "net.core.somaxconn = 551" >> /etc/sysctl.conf

* 确保系统中设置了**swap**（官方建议swap的大小和内存差不多大）。如果你没有设置swap，当内存使用过多时会内存溢出导致服务崩溃或Linux kernel OOM killer将可能会杀掉Redis进程

* 最好设置maxmemory参数来控制Redis最大使用内存

* 当你的Redis运行在daemontools下，请设置`daemonize`参数为`no`

* 即使已关闭Redis的持久化功能，当你使用主从复制时还是会执行`RDB saves`，除非你使用了`diskless replication`特性（此新特性还处于试验阶段）。

* 当你使用主从复制时，请确保master开启了持久化功能或者Redis遇错时不让它自动重启。因为当master自动重启时，其为empty data set（因为没有持久化），此时slave执行replication时，slave也会清空所有的数据。

* Redis默认不需要任何的身份验证且监听所有的网络接口，当你的Redis暴露在公网下面时会有很大的安全隐患。参考链接：<http://redis.io/topics/security>

### Redis配置

----------***缓存服务配置***--------------

1.Master需要修改配置

```bash
daemonize yes
#此值应该与对应的/etc/init.d/redis*的PIDFILE参数值相同
pidfile /var/run/redis_6379.pid
#此值应该与对应的/etc/init.d/redis*的端口号相同
port 6379
#根据服务器的内存配置自定义，需要预留部分内存
maxmemory 2gb
#超出最大指定内存时，删除最近最少使用的keys
maxmemory-policy allkeys-lru
#注释下面三个原有配置，因为此服务纯粹为缓存服务，不需要持久化数据，注释以提高性能
#save 900 1
#save 300 10
#save 60 10000
#Redis存放RDB、AOF的目录
dir /var/redis/6379
```

2.Slave需要修改配置

```bash
daemonize yes
#此值应该与对应的/etc/init.d/redis*的PIDFILE参数值相同
pidfile /var/run/redis_6379.pid
#此值应该与对应的/etc/init.d/redis*的端口号相同
port 6379
#根据服务器的内存配置自定义，需要预留部分内存
maxmemory 2gb
#超出最大指定内存时，删除最近最少使用的keys
maxmemory-policy allkeys-lru
#配置Master地址，可以是IP或域名
slaveof 192.168.1.1 6379
#注释下面三个原有配置，因为此服务纯粹为缓存服务，不需要持久化数据，注释以提高性能
#save 900 1
#save 300 10
#save 60 10000
#Redis存放RDB、AOF的目录
dir /var/redis/6379
```

----------***Session服务配置***--------------

1.Master需要修改配置

```bash
daemonize yes
#此值应该与对应的/etc/init.d/redis*的PIDFILE参数值相同
pidfile /var/run/redis_6379.pid
#此值应该与对应的/etc/init.d/redis*的端口号相同
port 6379
#根据服务器的内存配置自定义，需要预留部分内存
maxmemory 2gb
#因为存储的session，所以不能随意删除数据
maxmemory-policy noeviction
#如果你想数据高完整性，建议开启AOF功能
appendonly yes
#Redis存放RDB、AOF的目录
dir /var/redis/6379
```

2.Slave需要修改配置

```bash
daemonize yes
#此值应该与对应的/etc/init.d/redis*的PIDFILE参数值相同
pidfile /var/run/redis_6379.pid
#此值应该与对应的/etc/init.d/redis*的端口号相同
port 6379
#根据服务器的内存配置自定义，需要预留部分内存
maxmemory 2gb
#因为存储的session，所以不能随意删除数据
maxmemory-policy noeviction
#配置Master地址，可以是IP或域名
slaveof 192.168.1.2 6379
#Redis存放RDB、AOF的目录
dir /var/redis/6379
```

Redis安装目录会有个redis.conf文件，里面包含各种配置参数及讲解，非常详细，我觉得Redis这点做的非常好。配置文件的格式非常简单，参数名和参数值以空格隔开，如果想参数值包含字符窜，则参数值加上引号：`requirepass "hello world"`  

当然你也可以在启动服务时设置参数：`./redis-server --port 6380 --slaveof 127.0.0.1 6379`。在参数名前加上`--`就行了  

当Redis服务正在运行时，你同样可以设置参数，且不会影响Redis的运行。设置参数命令为`CONFIG SET`，获取参数命令为`CONFIG GET`。有一点需要注意，通过`CONFIG SET`修改的参数，在你重启服务时此参数不会生效，除非你同事修改了redis.conf文件。

以下列表的配置都是默认值：

* **maxmemory \<bytes\>**  
maxmemory值为0时，内存使用没有限制。64位系统默认为0，32位隐性设置为3GB。个人推荐明确配置maxmemory。由于内存回收机制的原因，如果你Redis内存使用的峰值为8G，但是绝大多数的情况下Redis只需要5G就满足了，这个时候你会发现内存占用一直是8G（有个专业词：Resident Set Size简称RSS），另外的3G并没有真正的腾出来。但是那3G的内存还是可以提供给你再次存放Redis数据的，这一点无需担心。可以通过设置maxmemory来限制最大使用内存，另外通过maxmemory-policy来配置当内容达到最大限制时是删除不常使用的key还是报错。  

当你没有设置maxmemory时，意味着你的Redis服务使用内存是无限制的，当超出服务器所能承受的最大使用内存时会影响服务甚至宕机。所以建议设置maxmemory，且这个值不能过于接近服务器最大使用内存。除非你非常确定你的Redis服务内存使用稳定，不会出现过大的峰值。

* **maxmemory-policy noeviction**  
	* volatile-lru 用lru算法剔除有过期限制的数据
	* allkeys-lru 用lru算法剔除任意的数据
	* volatile-random 随机剔除有过期限制的数据
	* allkeys-random 随机剔除任意的数据
	* volatile-ttl 剔除快要过期的数据
	* noeviction 不删除数据，当达到做大使用内存时报错  
	当策略为 volatile-lru、volatile-random、volatile-ttl，且没有可以删除的keys时，行为会和noeviction一样抛出错误。

* **maxmemory-samples 5**  
当需要剔除数据时，挑出5条数据依据maxmemory-policy策略删除一条。为什么每次只取样5个来删除数据呢？官方的解释是：这样性能更快。典型的拿准确率换性能。

具体请参考官方文档：<http://redis.io/topics/lru-cache>

### 运行Redis

运行以上命令后，Redis就安装、配置好了

运行Redis：

```bash
service redis_6379 start
```

使用Redis内嵌的命令行客户端：

```bash
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```
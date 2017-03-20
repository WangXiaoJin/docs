## RabbitMQ安装及配置

### Windows环境

1. 下载并安装Erlang的windows环境二进制可执行文件<http://www.erlang.org/downloads>

2. 下载并安装RabbitMQ程序<https://www.rabbitmq.com/download.html>

3. 请确保rabbitmq能绑定以下的端口，不然会启动失败：

	```
    4369 (epmd), 25672 (Erlang distribution)  
    5672, 5671 (AMQP 0-9-1 without and with TLS)  
    15672 (if management plugin is enabled)  
    61613, 61614 (if STOMP is enabled)  
    1883, 8883 (if MQTT is enabled)  
	```
	
	> 可通过修改配置文件来修改端口号，具体请往下看


### CentOS环境

1. 下确保已安装Erlang，下载地址：<https://www.rabbitmq.com/releases/erlang/>  

	> 注：如果你下载rpm包安装erlang时需要先安装它的依赖包：`yum install unixODBC unixODBC-devel wxBase wxGTK SDL wxGTK-gl`
	> 如果你下载的是`erlang-18.*`高版本，需要升级glibc到2.15，推荐下载`erlang-17.4-1.el6.x86_64.rpm`
	
	内置版本安装
    For Homebrew on OS X: brew install erlang  
    For MacPorts on OS X: port install erlang  
    For Ubuntu and Debian: apt-get install erlang  
    For Fedora: yum install erlang  
    For FreeBSD: pkg install erlang
	
2. 安装。下载rabbitmq <https://www.rabbitmq.com/download.html>，并执行以下命令：

	```bash
	rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
	yum install rabbitmq-server-*.*.*-*.noarch.rpm
	```

	> 如果安装时报`Requires: socat`异常，需要安装socat：
	
	```bash
	wget –no-cache http://www.convirture.com/repos/definitions/rhel/6.x/convirt.repo -O /etc/yum.repos.d/convirt.repo
	yum makecache
	yum install -y socat
	```

3. 启动服务

	```bash
	#开机自启服务
	chkconfig rabbitmq-server on
	#关闭、启动服务
	service rabbitmq-server stop/start
	```
	
	> 注：rabbitmq默认是用`rabbitmq`用户启动服务的，所以如果你改变了Mnesia database或logs文件的地址，请确保这些文件的拥有者是`rabbitmq`，让其有权限操作文件。
	
	请确保rabbitmq能绑定以下的端口，不然会启动失败：
	
	```
    4369 (epmd), 25672 (Erlang distribution)
    5672, 5671 (AMQP 0-9-1 without and with TLS)
    15672 (if management plugin is enabled)
    61613, 61614 (if STOMP is enabled)
    1883, 8883 (if MQTT is enabled)
	```
	
	> 中间件会创建一个【用户名：guest 密码：guest】的账号，默认这个账号只能只能被localhost使用，可修改配置文件。

4. 系统配置 

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
		

### 配置

1. 文件路径
	
    Generic UNIX - $RABBITMQ_HOME/etc/rabbitmq/  
    Debian - /etc/rabbitmq/  
    RPM - /etc/rabbitmq/  
    Mac OS X (Homebrew) - ${install_prefix}/etc/rabbitmq/, the Homebrew prefix is usually /usr/local  
    Windows - %APPDATA%\RabbitMQ\
	
	> 注：默认情况下配置文件没有创建。可手动创建rabbitmq-env.conf（windows文件名rabbitmq-env.bat）【环境变量配置文件】、rabbitmq.config，
	修改后需要重启服务，windows需要re-install service。  
	> `rabbitmq.config`配置文件有一个例子，文件地址：`/usr/share/doc/rabbitmq-server-*.*.*/rabbitmq.config.example`

	**Generic Unix Default Locations**
	
	|Name	| Location|
	| ------------- | ------------- |
	|RABBITMQ_BASE 	|(Not used)|
	|RABBITMQ_CONFIG_FILE 	|$RABBITMQ_HOME/etc/rabbitmq/rabbitmq|
	|RABBITMQ_MNESIA_BASE 	|$RABBITMQ_HOME/var/lib/rabbitmq/mnesia|
	|RABBITMQ_MNESIA_DIR 	|$RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME|
	|RABBITMQ_LOG_BASE 	|$RABBITMQ_HOME/var/log/rabbitmq|
	|RABBITMQ_LOGS 	|$RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME.log|
	|RABBITMQ_SASL_LOGS 	|$RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME-sasl.log|
	|RABBITMQ_PLUGINS_DIR 	|$RABBITMQ_HOME/plugins|
	|RABBITMQ_PLUGINS_EXPAND_DIR 	|$RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME-plugins-expand|
	|RABBITMQ_ENABLED_PLUGINS_FILE 	|$RABBITMQ_HOME/etc/rabbitmq/enabled_plugins|
	|RABBITMQ_PID_FILE 	|$RABBITMQ_MNESIA_DIR.pid|

	** 可以通过日志查看启动服务所用的配置文件路径：**
	
	```
	node           : rabbit@example
	home dir       : /var/lib/rabbitmq
	config file(s) : /etc/rabbitmq/rabbitmq.config
	```
	
	
	
	
	
	

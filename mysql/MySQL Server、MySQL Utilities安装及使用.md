## MySQL Server、MySQL Utilities安装及使用

##### `【原创】` :heart_eyes:
---

　　最近正在研究MySQl的高可用性、负载均衡、分库分表，借此机会记录下自己
粗陋的理解。

### Windows安装MySQL Server
Windows安装分为两种方式：一种是以msi结尾的可视化安装程序；另一种是免安装包

1. 可视化安装  
	下载地址：[http://dev.mysql.com/downloads/installer/](http://dev.mysql.com/downloads/installer/)。这种方式是最常见，且安装较简单。安装时自带可选工具：MySQL Workbench, MySQL Notifier, and MySQL for Excel。它能执行安装、更新、删除、配置MySQL。  
	安装文档：[http://dev.mysql.com/doc/refman/5.7/en/mysql-installer-gui.html](http://dev.mysql.com/doc/refman/5.7/en/mysql-installer-gui.html)

2. 免安装包  
	下载地址：[http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)。这种方式需要手动配置文件。  
	第一步：修改my.ini或者my.cnf配置文件
  
		[mysqld]
		# 设置MySQL的安装路径
		basedir=E:/mysql
		# 设置MySQL的Data目录
		datadir=E:/mydata/data
	第二步：[初始化MySQL的Data目录](#initDataDir)  
	第三步：开启MySQL服务

	* 直接启动
	
			C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqld"
	* 添加MySQL服务，通过服务来管理MySQL的自动启动  
		如果数据库（mysqld）已启动，则需要关闭数据库
		
			C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin"
          	-u root shutdown
		如果root密码已设置，则需要增加`-p`参数。  
		注册MySQL服务

			#开机即启动服务
			C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqld" --install
			#手动启动服务
			C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqld" --install-manual
		注销MySQL服务
			
			C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqld" --remove

	第三步：[修改初始密码](#updPwd)

### 配置MySQL Server  
参考：[配置文件](http://dev.mysql.com/doc/refman/5.7/en/option-files.html)  

1. Windows环境的配置文件路径，加载顺序为表格的顺序  

| File Name | Purpose  |
| :-------: | :------: |
| %PROGRAMDATA%\MySQL\MySQL Server 5.7\my.ini, %PROGRAMDATA%\MySQL\MySQL Server 5.7\my.cnf | Global options |
| %WINDIR%\my.ini, %WINDIR%\my.cnf | Global options |
| C:\my.ini, C:\my.cnf | Global options |
| INSTALLDIR\my.ini, INSTALLDIR\my.cnf | Global options |
| defaults-extra-file | The file specified with --defaults-extra-file=file_name, if any |
| %APPDATA%\MySQL\.mylogin.cnf | Login path options |

2. Unix, Linux and OS X环境的配置文件路径，加载顺序为表格的顺序  

| File Name | Purpose  |
| :-------: | :------: |
| /etc/my.cnf | Global options |
| /etc/mysql/my.cnf | Global options |
| SYSCONFDIR/my.cnf | Global options |
| $MYSQL_HOME/my.cnf | Server-specific options |
| defaults-extra-file | The file specified with --defaults-extra-file=file_name, if any |
| ~/.my.cnf | User-specific options |
| ~/.mylogin.cnf | Login path options |

### <a name="initDataDir"></a>初始化MySQL的Data目录  
Windows初始化Data目录命令：  

	C:\> bin\mysqld --initialize
	C:\> bin\mysqld --initialize-insecure

`--initialize`和`--initialize-insecure`的区别在于initialize会随机生成'root'@'localhost'账号的密码，且密码是可过期的。而initialize-insecure的密码还是为空。  
Linux初始化Data目录命令：  

	shell> bin/mysqld --initialize --user=mysql
	shell> bin/mysqld --initialize-insecure --user=mysql

确保数据库的文件和目录的拥有者为`mysql`用户，使其有读和写的权限。如果你登录当前系统使用的就是`mysql`用户则没必要加这个参数。

>#### 注意：
>以上命令适用于`MySQL 5.7.6`的所有操作系统。如果是Unix环境且版本在`5.7.6`之前，则使用`mysql_install_db`命令。
>如果是Windows环境且在`MySQL 5.7.7`版本之前，免安装版解压缩后会预包含Data目录，所以不需要执行上述命令。

执行初始化命令执行如下操作：  

1. 检测Data目录是否存在  
	* 如果Data目录不存在，则创建  
	* 如Data目录存在，且不为空，命令退出且出现错误信息。此时需要移除此目录后再操作
* 在Data目录下面创建系统库和系统表，包含grant tables、help tables、time zone tables等
* 初始化`system tablespace`和管理`InnoDB`表的数据结构
* 创建'root'@'localhost'超级管理员账号。依据`--initialize`、`--initialize-insecure`确定是否自动生成密码。

初始化Data目录后需要修改'root'@'localhost'密码，操作前提是MySQL服务已启动：

1. 启动MySQL服务，参考上面的安装步骤
2. 连接数据库
	* 如果你初始化的参数是`--initialize`
	
			shell> mysql -u root -p
			Enter password: (enter the random root password here)
	* 如果你初始化的参数是`--initialize-insecure`
	
			shell> mysql -u root --skip-password
3. 连接成功后修改root密码
	
		mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';

>注意：  
>[`初始化Data目录`官方文档](http://dev.mysql.com/doc/refman/5.7/en/data-directory-initialization-mysqld.html) 。
>修改密码可参考[修改初始密码](#updPwd)

### 修改初始密码  
<a name="updPwd"></a>
查看数据库中包含哪些账号，他们的密码是否为空：

```
#假设root账号的密码为空
shell> mysql -u root
```

```
#MySQL 5.7.6版本使用
mysql> SELECT User, Host, HEX(authentication_string) FROM mysql.user;
```

>注意：[`修改初始密码`官方文档](http://dev.mysql.com/doc/refman/5.7/en/default-privileges.html) 

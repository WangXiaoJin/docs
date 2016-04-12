## Disable Transparent Huge Pages (THP)

#####　翻译自MongoDB的官方文档<https://docs.mongodb.org/manual/tutorial/transparent-huge-pages/>
---

Transparent Huge Pages (THP)是Linux内存管理系统。当你的机器使用大量的larger memory pages时，THP能减少Translation Lookaside Buffer (TLB)查询性能的开销。
然而，当THP应用于数据库时，表现不佳。


1. 创建 init.d script  
创建/etc/init.d/disable-transparent-hugepages文件，内容如下：

	```bash
	#!/bin/sh
	### BEGIN INIT INFO
	# Provides:          disable-transparent-hugepages
	# Required-Start:    $local_fs
	# Required-Stop:
	# X-Start-Before:    mongod mongodb-mms-automation-agent
	# Default-Start:     2 3 4 5
	# Default-Stop:      0 1 6
	# Short-Description: Disable Linux transparent huge pages
	# Description:       Disable Linux transparent huge pages, to improve
	#                    database performance.
	### END INIT INFO
	
	case $1 in
	  start)
	    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
	      thp_path=/sys/kernel/mm/transparent_hugepage
	    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
	      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
	    else
	      return 0
	    fi
	
	    echo 'never' > ${thp_path}/enabled
	    echo 'never' > ${thp_path}/defrag
	
	    unset thp_path
	    ;;
	esac
	```

2. 赋予可执行权限
	```bash
	sudo chmod 755 /etc/init.d/disable-transparent-hugepages
	```

3. 启动服务  
	
	> service disable-transparent-hugepages start

4. 开机自启服务

	| Linux版本 | 命令          |
	| ------------- | ----------- |
	| Ubuntu and Debian      | sudo update-rc.d disable-transparent-hugepages defaults|
	| SUSE     | sudo insserv /etc/init.d/disable-transparent-hugepages |
	| Red Hat, CentOS, Amazon Linux, and derivatives | sudo chkconfig --add disable-transparent-hugepages |

5. 如果你使用了**tuned** 或者 **ktune**，你需要重新配置（**当系统为 Red Hat 或 CentOS 6+ 时**）  
当你使用**tuned** 或者 **ktune**时，你需要修改或新建一个profile来设置THP为`never`  

	* Red Hat/CentOS 6
		1. 创建一个新profile  
		
			> sudo cp -r /etc/tune-profiles/default /etc/tune-profiles/no-thp
			
		2. 编辑ktune.sh  
			编辑/etc/tune-profiles/no-thp/ktune.sh。添加如下代码到ktune.sh文件的start()代码块中且在`return 0`语句之前：
			
			> set_transparent_hugepages never
			
		3. 启用新建的profile
			
			>sudo tuned-adm profile no-thp

	* Red Hat/CentOS 7
		1. 创建一个新profile  
			创建一个tuned profile目录：
			
			> sudo mkdir /etc/tuned/no-thp
			
		2. 编辑tuned.conf  
			创建/etc/tuned/no-thp/tuned.conf文件。文件内包含：
			
			```bash
			[main]
			include=virtual-guest
			
			[vm]
			transparent_hugepages=never
			```
			
		3. 启用新建的profile
			
			>sudo tuned-adm profile no-thp

6. 测试  
	你可以通过如下命令来检查THP的是否关闭：

	```bash
	cat /sys/kernel/mm/transparent_hugepage/enabled
	cat /sys/kernel/mm/transparent_hugepage/defrag
	```
	
	在Red Hat Enterprise Linux、CentOS、或其他的Red Hat衍生品系统中，你可以替换成下面的命令：
	
	```bash
	cat /sys/kernel/mm/redhat_transparent_hugepage/enabled
	cat /sys/kernel/mm/redhat_transparent_hugepage/defrag
	```
	
	最终显示的结果应该为：
	
	> always madvise [never]
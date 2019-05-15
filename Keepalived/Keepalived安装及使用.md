## Keepalived安装及使用

1. 安装

	```bash
	# 安装依赖包
	shell> yum install curl gcc openssl-devel libnl3-devel net-snmp-devel
	shell> curl --progress http://keepalived.org/software/keepalived-1.2.15.tar.gz | tar xz
	shell> cd keepalived-1.2.15
	# 可以指定安装目录 --prefix=/usr/local/keepalived-1.2.15
	# --sysconfdir 指定配置文件模板的安装路径，默认为 PREFIX/etc
	shell> ./configure --sysconfdir=/etc
	shell> make
	shell> sudo make install
	# create an init script in order to control the keepalived daemon
	# The link should be added in your default run level directory
	shell> ln -s /etc/rc.d/init.d/keepalived /etc/rc.d/rc3.d/S99keepalived
	```
	
	配置文件地址：
	* /etc/keepalived/keepalived.conf
	* /etc/keepalived/samples （里面很多配置例子 - 【重要】）
	* /etc/sysconfig/keepalived （启动项配置）
	* /etc/rc.d/init.d/keepalived
	
	注：[参考文档](http://www.keepalived.org/doc/installing_keepalived.html)

2. 配置

    修改`/etc/keepalived/keepalived.conf`配置文件
    
    `MASTER`配置：
    ```
    global_defs {
        notification_email {
            admin1@example1.com
            admin2@example1.com
        }
        notification_email_from keepalived@example1.com
        smtp_server 192.168.200.20
        smtp_connect_timeout 30
        # 通常设置成Hostname，随意设置，只需确保唯一
        router_id LVS_MASTER
    }
    vrrp_script chk_nginx {
        script "killall -0 nginx"       # cheaper than pidof
        interval 2                      # check every 2 seconds
        weight -4                       # default prio: -4 if KO
        fall 2                          # require 2 failures for KO
        rise 2                          # require 2 successes for OK
    }
    
    # 或者可以通过http_port来判断服务是否正常
    #vrrp_script chk_http_port {
    #    script "</dev/tcp/127.0.0.1/80" # connects and exits
    #    interval 1                      # check every second
    #    weight -2                       # default prio: -2 if connect fails
    #}
    
    vrrp_instance VI_1 {
        interface eth0  #配置自己的网卡
        state MASTER    # MASTER
        virtual_router_id 50    #取值范围0..255，在同一个网卡中vrrp实例的此值不可重复
        priority 100
        authentication {
            auth_type PASS
            # 密码仅前8个字符有效
            auth_pass 123qwe
        }
        virtual_ipaddress {
            192.168.200.100
        }
        track_script {
            chk_nginx
        }
    }
    ```
    
    `BACKUP`配置：
    ```
    global_defs {
        notification_email {
            admin1@example1.com
            admin2@example1.com
        }
        notification_email_from keepalived@example1.com
        smtp_server 192.168.200.20
        smtp_connect_timeout 30
        # 通常设置成Hostname，随意设置，只需确保唯一
        router_id LVS_BACKUP
    }
    vrrp_script chk_nginx {
        script "killall -0 nginx"       # cheaper than pidof
        interval 2                      # check every 2 seconds
        weight -4                       # default prio: -4 if KO
        fall 2                          # require 2 failures for KO
        rise 2                          # require 2 successes for OK
    }
    
    # 或者可以通过http_port来判断服务是否正常
    #vrrp_script chk_http_port {
    #    script "</dev/tcp/127.0.0.1/80" # connects and exits
    #    interval 1                      # check every second
    #    weight -2                       # default prio: -2 if connect fails
    #}
    
    vrrp_instance VI_1 {
        interface eth0  # 配置自己的网卡
        state BACKUP    # BACKUP
        virtual_router_id 50    #取值范围0..255，在同一个网卡中vrrp实例的此值不可重复
        priority 99
        authentication {
            auth_type PASS
            # 密码仅前8个字符有效
            auth_pass 123qwe
        }
        virtual_ipaddress {
            192.168.200.100
        }
        track_script {
            chk_nginx
        }
    }
    ```
    
    ```
    # 通过HTTP请求进行健康检查
    HTTP_GET {
        url {
            path /testurl/test.jsp
            digest ec90a42b99ea9a2f5ecbe213ac9eba03
        }
        connect_timeout 3
        retry 3
        delay_before_retry 2
    }
    ```
    > 注：url的`digest`值通过`genhash –s 192.168.100.2 –p 80 –u /testurl/test.jsp`命令生成
    
    文档：
    * `man keepalived.conf`/`man genhash`/`man keepalived` 查看文档，源文件位于源码包`doc/man/`目录下
    * [Keepalived configuration synopsis](http://www.keepalived.org/doc/configuration_synopsis.html)
    * [Case Study: Healthcheck](http://www.keepalived.org/doc/case_study_healthcheck.html)
    * [vrrp_script|track_script配置 - 【重要】](https://github.com/acassen/keepalived/blob/master/doc/samples/keepalived.conf.vrrp.localcheck)
	
3. 启动

	```bash
	shell> service keepalived start
	```
	
	注：[参考文档](http://www.keepalived.org/doc/programs_synopsis.html)

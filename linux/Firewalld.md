## 防火墙

### CentOS7 - Firewall

1. 开放防火墙端口

    列出指定zone的开放端口（不会显示`--add-services`开放的端口）:
	```bash
	shell> firewall-cmd --zone=public --list-ports
	```
	
	`public` zone开放端口：
    ```bash
    # 立即生效，firewalld reload、service restart或系统重启后会丢失此配置
    shell> firewall-cmd --zone=public --add-port=8080/tcp
 
    # 持久化配置，不会立即生效，firewalld reload、service restart或系统重启后才会生效
    shell> firewall-cmd --zone=public --add-port=8080/tcp --permanent
 
    # 扩大端口范围
    shell> firewall-cmd --zone=public --add-port=5060-5061/udp
    ```
    
    > 注：如果命令中没有`--zone=`参数则使用默认zone，默认zone可配置。默认zone为`public`,所以上述的`--zone=`参数可省略。

1. 显示防火墙的状态

	```bash
	shell> firewall-cmd --state
	```

1. 显示当前已激活`zones`的`interfaces`

    ```bash
    shell> firewall-cmd --get-active-zones
    public
       interfaces: em1
    ```

1. 显示指定`zone`的`interfaces`

    ```bash
    shell> firewall-cmd --zone=public --list-interfaces
    ```

1. 显示指定`zone`的所有配置

    ```bash
    shell> firewall-cmd --zone=public --list-all
    ```

1. 显示指定`zone`的信息，加上`-v`显示详情信息

    ```bash
    shell> firewall-cmd --info-zone=public -v
    ```

1. 显示当前已经加载的`services`

    ```bash
    shell> firewall-cmd --get-services
    ```

1. 显示持久化的`services`

    ```bash
    shell> firewall-cmd --permanent --get-services
    ```

1. 显示`service`配置

    ```bash
    shell> firewall-cmd --info-service=ftp
    ```

1. reload firewall
    
    reload firewall不会中断用户连接（不丢失状态信息），在reload阶段Service可能会被破坏
    ```bash
    shell> firewall-cmd --reload
    ```

> 注：[Firewall - 官方文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/security_guide/index#sec-Using_Firewalls)

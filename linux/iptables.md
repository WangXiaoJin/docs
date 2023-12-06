# iptables

+ iptables是一个配置Linux内核防火墙的命令行工具，是netfilter项目的一部分。术语iptables也经常代指该内核级防火墙。iptables可以直接配置，也可以通过许多前端和图形界面配置。iptables用于ipv4，ip6tables用于ipv6。nftables已经包含在Linux kernel 3.13中，以后会取代iptables成为主要的Linux防火墙工具。
+ iptables可以检测、修改、转发、重定向和丢弃IPv4数据包。过滤IPv4数据包的代码已经内置于内核中，并且按照不同的目的被组织成**表**的集合。**表**由一组预先定义的**链**组成，**链**包含遍历顺序规则。每一条规则包含一个谓词的潜在匹配和相应的动作（称为目标），如果谓词为**真**，该动作会被执行。也就是说**条件匹配**。iptables是用户工具，允许用户使用**链**和**规则**。

## 基本模块

### 表（Tables）

+ iptables 包含`5`张表（tables）
+ `raw`用于配置数据包，raw中的数据包不会被系统跟踪。
+ `filter`是用于存放所有与防火墙相关操作的默认表。
+ `nat`用于网络地址转换。
+ `mangle`用于对特定数据包的修改。
+ `security`用于强制访问控制网络规则。

### 链（Chains）
表由链组成，链是一些按顺序排列的规则的列表。默认的filter表包含INPUT，OUTPUT和FORWARD 3条内建的链，这3条链作用于数据包过滤过程中的不同时间点。默认情况下，任何链中都没有规则。可以向链中添加自己想用的规则。链的默认规则通常设置为 ACCEPT，如果想确保任何包都不能通过规则集，那么可以重置为DROP。默认的规则总是在一条链的最后生效，所以在默认规则生效前数据包需要通过所有存在的规则。

### 规则（Rules）
数据包的过滤基于规则。规则由一个目标（数据包包匹配所有条件后的动作）和很多匹配（导致该规则可以应用的数据包所满足的条件）指定。一个规则的典型匹配事项是数据包进入的端口（例如：eth0 或者 eth1）、数据包的类型（ICMP, TCP, 或者 UDP）和数据包的目的端口。目标使用-j或者--jump选项指定。目标可以是用户定义的链（例如，如果条件匹配，跳转到之后的用户定义的链，继续处理）、一个内置的特定目标或者是一个目标扩展。内置目标是 ACCEPT， DROP， QUEUE和RETURN，目标扩展是 REJECT and LOG。如果目标是内置目标，数据包的命运会立刻被决定并且在当前表的数据包的处理过程会停止。如果目标是用户定义的链，并且数据包成功穿过第二条链，目标将移动到原始链中的下一个规则。目标扩展可以被终止（像内置目标一样）或者不终止（像用户定义链一样）。

## 指令参数

### 查看配置
```
[root@vm-tongchen-test ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
```
+ 如上例所示，每一个数据包都要通过三个内建的链（INPUT、OUTPUT和FORWARD）中的一个。
+ filter是最常用的表，在filter表中最常用的三个目标是ACCEPT、DROP和REJECT。ACCEPT表示允许数据包通过，DROP则会丢弃数据包，不再对其进行任何处理。REJECT会把出错信息传送至发送数据包的主机。

> 上面示例为默认filter配置，如需显示其他表配置通过 -t 参数指定表明即可，例如显示nat表：
```
[root@vm-tongchen-test ~]# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DNAT       udp  --  anywhere             vm-tongchen-test     udp dpt:raid-ac to:192.168.66.66:2012
DNAT       tcp  --  anywhere             vm-tongchen-test     tcp dpt:ttyinfo to:192.168.66.66:2012
DNAT       tcp  --  anywhere             vm-tongchen-test     tcp dpt:51413 to:192.168.66.66:51413
DNAT       udp  --  anywhere             vm-tongchen-test     udp dpt:51413 to:192.168.66.66:51413

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  anywhere             anywhere            
```

iptables [-t table] COMMAND chain CRETIRIA -j ACTION

+ -t table ：3个filter nat mangle
+ COMMAND：定义如何对规则进行管理
+ chain：指定你接下来的规则到底是在哪个链上操作的，当定义策略的时候，是可以省略的
+ CRETIRIA:指定匹配标准
+ -j ACTION :指定如何进行处理

### 参数说明

|   参数    |   说明      |   示例      |
|:--------: |:----------: |:-----------:|
|    -F     | 清空规则链  | iptables -F |
|    -L 	| 查看规则链  | iptables -L |
|    -A 	|  追加规则   | iptables -A INPUT |
|    -D  	|  删除规则   |	iptables -D INPUT 1 |
|    -R 	|  修改规则   | iptable -R INPUT 1 -s 192.168.120.0 -j DROP |
|    -I 	| 在头部插入规则 | iptables -I INPUT 1 --dport 80 -j ACCEPT |
|    -L 	|  查看规则   |	iptables -L INPUT |
|    -N 	|  新的规则   | iptables -N allowed |
|    -V 	| 查看iptables版本 	| iptables -V|
|    -p 	| 协议（tcp/udp/icmp）| iptables -A INPUT -p tcp |
|    -s 	| 匹配原地址，加" ! "表示除这个IP外 | iptables -A INPUT -s 192.168.1.1 |
|    -d 	| 匹配目的地址 	| iptables -A INPUT -d 192.168.12.1 |
|  --sport 	| 匹配源端口流入的数据 	|iptables -A INPUT -p tcp --sport 22 |
|  --dport  | 匹配目的端口流出的数据|iptables -A INPUT -p tcp --dport 22 |
|    -i 	| 匹配入口网卡流入的数据 | iptables -A INPUT -i eth0 |
|    -o 	| 匹配出口网卡流出的数据 | iptables -A FORWARD -o eth0 |
|    -j 	| 要进行的处理动作:DROP(丢弃)，REJECT(拒绝)，ACCEPT(接受)，SANT(基于原地址的转换) | iptable -A INPUT 1 -s 192.168.120.0 -j DROP |
| --to-source |	指定SANT转换后的地址 | iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -j SANT --to-source 172.16.100.1 |
|    -t 	| 表名(raw、mangle、nat、filter)| iptables -t nat |
|    -m 	| 使用扩展模块来进行数据包的匹配(multiport/tcp/state/addrtype) | iptables -m multiport |

### 动作说明

处理动作除了 ACCEPT、REJECT、DROP、REDIRECT 和 MASQUERADE 以外，还多出 LOG、ULOG、DNAT、SNAT、MIRROR、QUEUE、RETURN、TOS、TTL、MARK 等，其中某些处理动作不会中断过滤程序，某些处理动作则会中断同一规则链的过滤，并依照前述流程继续进行下一个规则链的过滤，一直到堆栈中的规则检查完毕为止。透过这种机制所带来的好处是，我们可以进行复杂、多重的封包过滤，简单的说，iptables 可以进行纵横交错式的过滤（tables）而非链状过滤（chains）。

| 动作 | 说明 | 示例 |
|:------:|:----:|:----:|
|ACCEPT|将封包放行，进行完此处理动作后，将不再比对其它规则，直接跳往下一个规则链（nat:postrouting）|  |
|REJECT|拦阻该封包，并传送封包通知对方。以传送的封包有几个选择：ICMP port-unreachable、ICMP、echo-reply或是tcp-reset（这个封包会要求对方关闭联机），进行完此处理动作后，将不再比对其它规则，直接中断过滤程序。|iptables -A FORWARD -p TCP --dport 22 -j REJECT --reject-with tcp-reset|
|DROP|丢弃封包不予处理，进行完此处理动作后，将不再比对其它规则，直接中断过滤程序。|    |
|REDIRECT|将封包重新导向到另一个端口（PNAT），进行完此处理动作后，将 会继续比对其它规则。这个功能可以用来实作通透式 porxy 或用来保护 web 服务器。|iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080|
|MASQUERADE|改写封包来源 IP 为防火墙 NIC IP，可以指定 port 对应的范围，进行完此处理动作后，直接跳往下一个规则链（mangle:postrouting）。这个功能与 SNAT 略有不同，当进行 IP 伪装时，不需指定要伪装成哪个 IP，IP 会从网卡直接读取，当使用拨接连线时，IP 通常是由 ISP 公司的 DHCP 服务器指派的，这个时候 MASQUERADE 特别有用。|iptables -t nat -A POSTROUTING -p TCP -j MASQUERADE --to-ports 1024-31000|
|LOG|将封包相关讯息纪录在 /var/log 中，详细位置请查阅 /etc/syslog.conf 组态档，进行完此处理动作后，将会继续比对其它规则。|iptables -A INPUT -p tcp -j LOG --log-prefix "INPUT packets"|
|SNAT|改写封包来源 IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将直接跳往下一个规则链（mangle:postrouting）。|iptables -t nat -A POSTROUTING -p tcp-o eth0 -j SNAT --to-source 194.236.50.155-194.236.50.160:1024-32000|
|DNAT|改写封包目的地 IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将会直接跳往下一个规则链（filter:input 或 filter:forward）。|iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.1.1-192.168.1.10:80-100|
|MIRROR|镜射封包，也就是将来源IP与目的地IP对调后，将封包送回，进行完此处理动作后，将会中断过滤程序。|   |
|QUEUE|中断过滤程序，将封包放入队列，交给其它程序处理。透过自行开发的处理程序，可以进行其它应用，例如：计算联机费用……等。|   |
|RETURN|结束在目前规则炼中的过滤程序，返回主规则炼继续过滤，如果把自订规则炼看成是一个子程序，那么这个动作，就相当于提早结束子程序并返回到主程序中。|	  |
|MARK|将封包标上某个代号，以便提供作为后续过滤的条件判断依据，进行完此处理动作后，将会继续比对其它规则。|iptables -t mangle -A PREROUTING -p tcp --dport 22 -j MARK --set-mark 2|


## 注意事项

### 打印跟踪日志

```shell
# 在 filter 表 FORWARD 链的最前面追加日志跟踪
shell> iptables -I FORWARD  -j LOG --log-prefix "FORWARD LOG: "
```

> 注：链中的日志操作不会影响后面的规则执行。如果想判断某规则是否成功匹配并执行，在当前规则前后各增加日志，
> 1. 规则之前的日志打印，说明iptables执行经过此规则。
> 2. 规则之后的日志打印，说明此规则不匹配，否则说明此规则生效。
> 3. 查看日志方案：a. `tail -f /var/log/messages` b. `journalctl -kf`

### 启用 ip_forward

当使用`nat`表中的`PREROUTING`链时需要配置ip_forward，如下：
```shell
shell> /etc/sysctl.conf 设置 net.ipv4.ip_forward = 1
shell> sysctl -p
```

> 注：`nat`表的`OUTPUT`链无需启用`ip_forward`，因为请求是从当前主机发起的。



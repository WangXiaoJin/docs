## Dubbo

### 协议参考手册

* dubbo 协议
    * 为什么要消费者比提供者个数多?
    * 为什么不能传大包?
    * 为什么不能传大包?
* rest 协议
* http 协议
* hessian 协议
* redis 协议
* thrift 协议
* gRPC 协议
* memcached 协议
* rmi 协议
* webservice 协议

> [Dubbo 2.7 协议参考手册](http://dubbo.apache.org/zh/docs/v2.7/user/references/protocol/)

### dubbo-monitor

dubbo-monitor统计生效的前提：
* dubbo-monitor版本号和dubbo版本号最好一致，否则及有可能不生效
* dubbo-monitor配置项`dubbo.jetty.directory`、`dubbo.charts.directory`、`dubbo.statistics.directory`的目录需要自己手动创建，
`charts`、`statistics`除外
* `dubbo provider/consumer`需要增加配置：`<dubbo:monitor protocol="registry"/>`

> 注：定时删除`charts`、`statistics`目录中的数据，否则磁盘占用会越来越大



## Dubbo

#### dubbo-monitor

dubbo-monitor统计生效的前提：
* dubbo-monitor版本号和dubbo版本号最好一致，否则及有可能不生效
* dubbo-monitor配置项`dubbo.jetty.directory`、`dubbo.charts.directory`、`dubbo.statistics.directory`的目录需要自己手动创建，
`charts`、`statistics`除外
* `dubbo provider/consumer`需要增加配置：`<dubbo:monitor protocol="registry"/>`

> 注：定时删除`charts`、`statistics`目录中的数据，否则磁盘占用会越来越大



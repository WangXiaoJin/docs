# MySql-常用

## 解答

### 锁

* [Select for update使用详解](https://zhuanlan.zhihu.com/p/143866444) - `主键`/`索引`/`非索引`字段 `行锁`及`表锁`场景
* [15.7.3 Locks Set by Different SQL Statements in InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html) - 官网

以下链接有待确定：
* [mysql 查看死锁和去除死锁](https://www.cnblogs.com/duanxz/p/4394641.html)
* [mysql insert锁机制](https://blog.csdn.net/zhanghongzheng3213/article/details/53436240)
* [全面了解mysql锁机制（InnoDB）与问题排查](https://juejin.im/post/5b82e0196fb9a019f47d1823)

## 索引

* [mysql索引结构原理、性能分析与优化](http://wulijun.github.io/2012/08/21/mysql-index-implementation-and-optimization.html)
* [《高性能MySQL》读后感——聚簇索引](https://www.jianshu.com/p/54c6d5db4fe6)

## 规范

* [mysql字段定义不要用null的分析](https://www.cnblogs.com/balfish/p/7905100.html)
* [Problems with NULL Values](https://dev.mysql.com/doc/refman/5.6/en/problems-with-null.html)

## Binlog

```shell
# 解析Binlog文件，加上 -d {db-name} 参数后部分SQL可能会不显示。可借助 `-d` 初步快速筛选Pos。
> mysqlbinlog --base64-output=decode-rows -vv --start-datetime='2022-06-14 14:00:00' --stop-datetime='2022-06-14 15:00:00' mysql-bin.000606 > /tmp/xx.sql
```
* [mysql查看binlog日志](https://www.cnblogs.com/softidea/p/12624778.html)
* [4.6.7 mysqlbinlog — Utility for Processing Binary Log Files](https://dev.mysql.com/doc/refman/5.7/en/mysqlbinlog.html)
* [4.6.7.2 mysqlbinlog Row Event Display](https://dev.mysql.com/doc/refman/5.7/en/mysqlbinlog-row-events.html)

## XA

* xid 格式 - `gtrid [, bqual [, formatID ]]`
* `gtrid` - a global transaction identifier
* `bqual` - a branch qualifier，默认值`''`
* `formatID` - a number that identifies the format used by the gtrid and bqual values，默认值`1`

> 具体规范参考[官方文档](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)

**XA RECOVER;（查看当前处于 PREPARED 状态的 XA 事务）**

|     formatID     |  gtrid_length  | bqual_length |   data   |
|:-------------:|:-----------:|:------------:|:--------:|
|  9752  |  33 |       19       | 10.90.36.5:8091:45309281615646740-333537489298999625 |

* `data`前33位字符为`gtrid`
* `data`后19位字符为`bqual`
* xid为：`'10.90.36.5:8091:45309281615646740','-333537489298999625',9752`

**XA RECOVER CONVERT xid;（XID 转换为 16 进制数据）**

|     formatID     |  gtrid_length  | bqual_length |   data   |
|:-------------:|:-----------:|:------------:|:--------:|
|  9752  |  33 |       19       | 0x31302E39302E33362E353A383039313A34353330393238313631353634363734302D333333353337343839323938393939363235 |

* `data`前`33*2=66`位字符为`gtrid`
* `data`后`19*2=38`位字符为`bqual`
* xid 16进制：`X'31302e39302e33362e353a383039313a3435333039323831363135363436373430',X'2d333333353337343839323938393939363235',9752`
> 因使用16进制输出数据，data字段值长度翻倍，**需注意大小写格式**

> 参考：[MySQL 分布式事务的“路”与“坑”](https://juejin.cn/post/7075526192006168607)


### 参考文档
* [13.3.7 XA Transactions](https://dev.mysql.com/doc/refman/5.7/en/xa.html)
* [XA 事务及在Seata中的应用](https://segmentfault.com/a/1190000040564227)
* [无处不在的 MySQL XA 事务](https://zhuanlan.zhihu.com/p/372300181)


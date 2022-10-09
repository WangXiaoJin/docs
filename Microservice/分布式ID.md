# 分布式ID

## Snowflake - 雪花算法

* 优点:
  * 生成速度快
  * 实现简单,没有多余的依赖
  * 可以根据实际情况调整各个位段,方便灵活
* 缺点:
  * 只能趋势递增
  * 依赖机器时间. 如果发生回拨可能会造成生成的ID重复
  * 前后端交互接口需要注意**精度丢失**问题。

> JS只有`Number`这一种数值类型，用`64-bit IEEE 754`双精度浮点数存储，这意味着不仅仅小数会丢失精度，**大于9007199254740991（16位数字）的整数** /`2^53-1`/`Number.MAX_SAFE_INTEGER`
也会存在丢失精度的风险。而雪花算法的Long值都非常大，很容易出现精度丢失问题，需要后端把数值转换成字符串类型，可借助于`Jackson`的`WRITE_NUMBERS_AS_STRINGS: true`功能实现。

### 时间回拨

* 产生原因:
  * 机器需要同步时间服务器
* 解决方案:
  * 当回拨时间小于X,可以等待时间追上来以后再继续生成
  * 当回拨时间大于X 时可以通过更换workId来产生之前都没有产生过的Id来解决回拨问题

### 参考

* [雪花算法：分布式唯一ID生成利器](https://segmentfault.com/a/1190000041445831)
* Seata
  * [Seata基于改良版雪花算法的分布式UUID生成器分析](https://seata.io/zh-cn/blog/seata-analysis-UUID-generator.html)
  * [Seata - 关于新版雪花算法的答疑](https://seata.io/zh-cn/blog/seata-snowflake-explain.html)
* [雪花算法把玩](http://yang.observer/2020/08/30/snowflake/)
  * 各场景下WorkerID如何生成
  * 系统时间回退怎么办
* [多时钟 解决雪花算法的时间回拨问题](https://blog.hackerpie.com/posts/algorithms/snowflake/multiple-clocks-snowflake/)
* [SnowFlake 雪花算法生成分布式 ID](http://learn.lianglianglee.com/%E6%96%87%E7%AB%A0/SnowFlake%20%E9%9B%AA%E8%8A%B1%E7%AE%97%E6%B3%95%E7%94%9F%E6%88%90%E5%88%86%E5%B8%83%E5%BC%8F%20ID.md) - 参考实现

## Butterfly 

雪花算法的改良方案，**思路可作参考**。

> [Butterfly 官网](https://www.yuque.com/simonalong/butterfly/tul824)


## UidGenerator

UidGenerator 基于Snowflake算法的Java唯一ID生成器。UidGenerator以组件形式工作在应用项目中, 支持自定义workerId位数和初始化策略, 
从而适用于docker等虚拟化环境下实例自动重启、漂移等场景。 在实现上, UidGenerator通过借用未来时间来解决sequence天然存在的并发限制; 
采用RingBuffer来缓存已生成的UID, 并行化UID的生产和消费, 同时对CacheLine补齐，避免了由RingBuffer带来的硬件级「伪共享」问题. 最终单机QPS可达600万。

* [UidGenerator 文档](https://github.com/baidu/uid-generator/blob/master/README.zh_cn.md)

> 注：UidGenerator 依赖DB 生产 WorkerId


## Leaf

Leaf是美团基础研发平台推出的一个分布式ID生成服务。

Leaf 提供两种生成的ID的方式（号段模式和snowflake模式），你可以同时开启两种方式，也可以指定开启某种方式（默认两种方式为关闭状态）。

* [Leaf——美团点评分布式ID生成系统](https://tech.meituan.com/2017/04/21/mt-leaf.html)
* [Leaf：美团分布式ID生成服务开源](https://tech.meituan.com/2019/03/07/open-source-project-leaf.html)
* [使用文档 - Github](https://github.com/Meituan-Dianping/Leaf/blob/master/README_CN.md)
* [Spring-Boot-Start](https://github.com/Meituan-Dianping/Leaf/blob/feature/spring-boot-starter/README_CN.md)


## Tinyid

Tinyid是用Java开发的一款分布式id生成系统，基于数据库号段算法实现，关于这个算法可以参考美团leaf或者tinyid原理介绍。Tinyid扩展了leaf-segment算法，
支持了多db(master)，同时提供了java-client(sdk)使id生成本地化，获得了更好的性能与可用性。

* [概要 - Github](https://github.com/didi/tinyid/wiki)
* [Getting started](https://github.com/didi/tinyid/wiki/Getting-started)
* [Tinyid client config](https://github.com/didi/tinyid/wiki/Tinyid-client-config)
* [Tinyid server config](https://github.com/didi/tinyid/wiki/Tinyid-server-config)
* [Tinyid原理介绍](https://github.com/didi/tinyid/wiki/Tinyid%E5%8E%9F%E7%90%86%E4%BB%8B%E7%BB%8D)

## CosId -【推荐】

通用、灵活、高性能分布式ID生成器
* 支持多种类型的分布式ID算法：SnowflakeId、SegmentId、SegmentChainId。 并且支持多种号段分发器、机器号分发器。
* 通过简单配置即可自定义切换多种算法实现，定制以满足场景需要。
* 设计极致优化，SegmentChainId 性能可达到近似 AtomicLong 的 TPS 性能:12743W+/s。

> [官方文档](https://cosid.ahoo.me/guide/)

> [Github 文档](https://github.com/Ahoo-Wang/CosId) - 配置文档相对全面


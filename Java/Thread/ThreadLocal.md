# ThreadLocal

## Transmittable ThreadLocal（TTL）

三种解决方案：
1. 修饰Runnable和Callable
2. 修饰线程池
3. 使用Java Agent来修饰JDK线程池实现类

* [TTL - Github主页](https://github.com/alibaba/transmittable-thread-local)
    * TTL Agent与其它Agent（如Skywalking、Promethues）配合使用时不生效？

* [需求场景](https://github.com/alibaba/transmittable-thread-local/blob/master/docs/requirement-scenario.md)
  * 分布式跟踪系统
  * 日志收集记录系统上下文
    * Log4j2 MDC的TTL集成
    * Logback MDC的TTL集成
  * Session级Cache
  * 应用容器或上层框架跨应用代码给下层SDK传递信息

  
## 参考链接

* [ThreadLocal垮线程池传递数据解决方案：TransmittableThreadLocal【享学Java】](https://cloud.tencent.com/developer/article/1600754)















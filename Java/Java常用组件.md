# Java常用组件

## 1. 核心工具库

### Vavr - [链接](https://docs.vavr.io/)

Vavr (formerly called Javaslang) is a functional library for Java 8+ that provides persistent data types and functional control structures.

### Guava

* [GitHub 链接](https://github.com/google/guava)
* [Google Guava官方教程（中文版）](https://wizardforcel.gitbooks.io/guava-tutorial/content/1.html)

### LMAX Disruptor

The LMAX Disruptor is a high performance inter-thread messaging library. It provides a concurrent `ring buffer` data structure. 
It is designed to provide a low-latency, high-throughput work queue in `asynchronous event processing` architectures.

* [LMAX Disruptor: High performance alternative to bounded queues for exchanging data between concurrent threads](https://lmax-exchange.github.io/disruptor/disruptor.html) - 参考文献
* [LMAX Disruptor User Guide](https://lmax-exchange.github.io/disruptor/user-guide/index.html) - 用户手册
  * WaitStrategy种类及对比
    * `BlockingWaitStrategy`
    * `SleepingWaitStrategy`
    * `YieldingWaitStrategy`
    * `BusySpinWaitStrategy`
* 博客
  * [Dissecting the Disruptor: What's so special about a ring buffer?](http://mechanitis.blogspot.com/2011/06/dissecting-disruptor-whats-so-special.html)
  * [Dissecting the Disruptor: Writing to the ring buffer](http://mechanitis.blogspot.com/2011/07/dissecting-disruptor-writing-to-ring.html)
  * [Dissecting the Disruptor: Why it's so fast (part one) - Locks Are Bad](http://mechanitis.blogspot.com/2011/07/dissecting-disruptor-why-its-so-fast.html)
  * [Dissecting the Disruptor: How do I read from the ring buffer?](http://mechanitis.blogspot.com/2011/06/dissecting-disruptor-how-do-i-read-from.html)

## 2. JavaPoet

`JavaPoet` is a Java API for generating `.java` source files. 是`JavaWriter`的替代产品。

* [JavaPoet官网](https://github.com/square/javapoet)


## 3. AutoService

A configuration/metadata generator for `java.util.ServiceLoader-style` service providers.

Java annotation processors and other systems use `java.util.ServiceLoader` to register implementations of well-known
types using `META-INF metadata`. However, it is easy for a developer to forget to update or correctly specify the
service descriptors. AutoService generates this metadata for the developer, for any class annotated with `@AutoService`,
avoiding typos, providing resistance to errors from refactoring, etc.

* [AutoService官网](https://github.com/google/auto/tree/master/service)
* [Google AutoService](https://www.baeldung.com/google-autoservice)

## 4. Immutables

Java annotation processors to generate simple, safe and consistent value objects. Do not repeat yourself, try Immutables,
the most comprehensive tool in this field!

* [Immutables官网](http://immutables.github.io/)

## 5. HTTP

#### Spring RestTemplate
#### Apache HttpClient
* [How to send HTTP request GET/POST in Java](https://mkyong.com/java/how-to-send-http-request-getpost-in-java/)
#### OkHttp
* [How to send HTTP request GET/POST in Java](https://mkyong.com/java/how-to-send-http-request-getpost-in-java/)
#### Java11 HttpClient
* [How to send HTTP request GET/POST in Java](https://mkyong.com/java/how-to-send-http-request-getpost-in-java/)
#### JDK HttpUrlConnection
* [How to use java.net.URLConnection to fire and handle HTTP requests?](https://stackoverflow.com/questions/2793150/how-to-use-java-net-urlconnection-to-fire-and-handle-http-requests/2793153#2793153)
* [Do a Simple HTTP Request in Java](https://www.baeldung.com/java-http-request)
* [How to send HTTP request GET/POST in Java](https://mkyong.com/java/how-to-send-http-request-getpost-in-java/)


## 6. 对象转换

#### JSON
* [JsonUnit](https://github.com/lukas-krecan/JsonUnit) - Compare JSON in your Unit Tests

#### ModelMapper
ModelMapper is an intelligent object mapping library that automatically maps objects to each other. It uses a convention
based approach while providing a simple refactoring safe API for handling specific use cases.
> [ModelMapper官网](http://modelmapper.org/getting-started/)


## 7. XML

* [XMLUnit](https://www.xmlunit.org/) - Unit Testing XML for Java and .NET

## 8. DB

* [dynamic-datasource-spring-boot-starter](https://github.com/baomidou/dynamic-datasource-spring-boot-starter)

    `dynamic-datasource-spring-boot-starter` 是一个基于springboot的快速集成多数据源的启动器。

    * 数据源分组，适用于多种场景 纯粹多库 读写分离 一主多从 混合模式。
    * 内置敏感参数加密和启动初始化表结构schema数据库database。
    * 提供对Druid，Mybatis-Plus，P6sy，Jndi的快速集成。
    * 简化Druid和HikariCp配置，提供全局参数配置。
    * 提供自定义数据源来源接口(默认使用yml或properties配置)。
    * 提供项目启动后增减数据源方案。
    * 提供Mybatis环境下的 纯读写分离 方案。
    * 使用spel动态参数解析数据源，如从session，header或参数中获取数据源。（多租户架构神器）
    * 提供多层数据源嵌套切换。（ServiceA >>> ServiceB >>> ServiceC，每个Service都是不同的数据源）
    * 提供 不使用注解 而 使用 正则 或 spel 来切换数据源方案（实验性功能）。
    * 基于seata的分布式事务支持。

* MyBatis
    * [MyBatis Products](https://blog.mybatis.org/p/products.html)
        * [Generator](http://www.mybatis.org/generator/)
        * [Migrations](http://www.mybatis.org/migrations)
        * [MyBatis Dynamic SQL](https://mybatis.org/mybatis-dynamic-sql/docs/introduction.html) -SQL Generator for MyBatis and Spring JDBC Templates

## 9. 监控

### Jolokia 

Jolokia is remote JMX with JSON over HTTP.

Jolokia is a JMX-HTTP bridge giving an alternative to JSR-160 connectors. It is an agent based approach with support 
for many platforms. In addition to basic JMX operations it enhances JMX remoting with unique features like bulk requests 
and fine grained security policies.

* [官方文档](https://jolokia.org/reference/html/index.html)

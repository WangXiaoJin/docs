# Java常用组件

## 1. 核心工具库

### Vavr - [链接](https://docs.vavr.io/)

Vavr (formerly called Javaslang) is a functional library for Java 8+ that provides persistent data types and functional control structures.


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

## 5. HTTP Request

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

* [Seata](https://seata.io/zh-cn/docs/overview/what-is-seata.html)

    Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。
    Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

* MyBatis
    * [MyBatis Products](https://blog.mybatis.org/p/products.html)
        * [Generator](http://www.mybatis.org/generator/)
        * [Migrations](http://www.mybatis.org/migrations)
        * [MyBatis Dynamic SQL](https://mybatis.org/mybatis-dynamic-sql/docs/introduction.html) -SQL Generator for MyBatis and Spring JDBC Templates
# Spring Cloud

## Config

* [Spring Cloud Config Refresh Strategies](https://soshace.com/spring-cloud-config-refresh-strategies/)


## Feign

* [SpringBoot2.1.x后Feign出现Bean被重复注册，导致项目不能启动](https://www.jianshu.com/p/b5581826cf67)


## Spring Cloud Gateway

#### Reactor Netty Access Logs

* [Reactor Netty Access Logs](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#reactor-netty-access-logs)

#### Log Levels

The following loggers may contain valuable troubleshooting information at the `DEBUG` and `TRACE` levels:
* `org.springframework.cloud.gateway`
* `org.springframework.http.server.reactive`
* `org.springframework.web.reactive`
* `org.springframework.boot.autoconfigure.web`
* `reactor.netty`
* `redisratelimiter`

#### Wiretap

The Reactor Netty `HttpClient` and `HttpServer` can have wiretap enabled. When combined with setting the `reactor.netty` 
log level to `DEBUG` or `TRACE`, it enables the logging of information, such as headers and bodies sent and received across the wire. 
To enable wiretap, set `spring.cloud.gateway.httpserver.wiretap=true` or `spring.cloud.gateway.httpclient.wiretap=true` for the `HttpServer` and `HttpClient`, respectively.



### 官方文档

* [Spring Cloud `CircuitBreaker` GatewayFilter Factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#spring-cloud-circuitbreaker-filter-factory)
* [The `RequestRateLimiter` GatewayFilter Factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory)
* [The `SaveSession` GatewayFilter Factory](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-savesession-gatewayfilter-factory)

## 问题

### `bootstrap` VS `spring.config.import`

`Spring Boot 2.4`/`spring cloud 3.x` 默认使用禁用了`bootstrap`机制加载配置，改用`spring.config.import`，
导致本地的`profiles`配置优先级高于配置中心的`profiles`配置。

暂时解决方案：
1. `spring.config.import=consul:`作为环境变量或启动参数
2. 提供一个高优先级(`Ordered.HIGHEST_PRECEDENCE`)的`EnvironmentPostProcessor`，把`spring.config.import=consul:`添加进`MapPropertySource`
3. 依赖`spring-cloud-starter-bootstrap`，启用`bootstrap`加载机制

参考链接:
* [The priority of defaulContext is now higher than the application context with spring.config.import](https://github.com/spring-cloud/spring-cloud-consul/issues/702)
* [ConfigData imports cannot override profile specific imports](https://github.com/spring-projects/spring-boot/issues/25766)
* [Ordering of remote vs local configuration files when using profiles is different between bootstrap and spring.config.import](https://github.com/spring-cloud/spring-cloud-config/issues/1795)



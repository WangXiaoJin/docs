# Spring Cloud

## Config

* [Spring Cloud Config Refresh Strategies](https://soshace.com/spring-cloud-config-refresh-strategies/)


## Feign

* [SpringBoot2.1.x后Feign出现Bean被重复注册，导致项目不能启动](https://www.jianshu.com/p/b5581826cf67)


## Spring Cloud Consul
Nginx配置：
```
upstream consul {
    ip_hash;
    server 192.168.201.1:8500 max_fails=10;
    server 192.168.201.2:8500 max_fails=10;
    server 192.168.201.3:8500 max_fails=10;
}

server {
    listen 80;
    server_name consul.xxx.com;
    # 第二个参数用于设置响应头Keep-Alive
    keepalive_timeout 75s 72s;
    proxy_ignore_client_abort off;
    proxy_next_upstream error timeout;
    
    location / {
        proxy_pass http://consul;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

问题汇总：
* `org.apache.http.NoHttpResponseException: The target server failed to respond` - [记HttpClient的NoHttpResponse问题](https://czjxy881.github.io/java,nginx/%E8%AE%B0HttpClient%E7%9A%84NoHttpResponse%E9%97%AE%E9%A2%98/)


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



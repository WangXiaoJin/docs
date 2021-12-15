# Spring Boot

### 当使用YAML格式配置日志时，你可能会遇到配置的OFF日志级别无效，因为YAML会把`OFF`/`ON`解析成`Boolean`类型的`false`/`true`。

**无效配置**
```yaml
logging:
level:
  root: info
  com.netflix.discovery: OFF
```

**有效配置**:
```yaml
logging:
level:
  root: info
  com.netflix.discovery: 'OFF'
```

### 动态修改配置

```
POST http://dev.bone-cloud-zuul.banksteel.com/actuator/env
{
    "name": "logging.level.org.springframework.security",
    "value": "debug"
}
```

### 开启`tomcat accesslog`

```yaml
server:
  tomcat:
    # 指定tomcat的basedir，没有指定则使用系统临时目录
    basedir: "/data/tomcat"
    accesslog:
      enabled: true
```

### Yaml Map Key 包含特殊字符，用方括号括起来

```yaml
spring:
  cors:
    cors-configurations:
      "[/api/**]":
        allowed-origins: "*"
        allowed-methods: [GET, HEAD, POST]
```
> [Escaping a dot in a Map key in Yaml in Spring Boot - 解决YAML Key包含特殊字符问题](https://stackoverflow.com/questions/34070987/escaping-a-dot-in-a-map-key-in-yaml-in-spring-boot/34082891#34082891)

### SpringBoot-2.4 `spring.config.import`

> [Config file processing in SpringBoot2.4](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)


### `spring.factories`中`EnableAutoConfiguration`加载顺序

1. 通过`ClassLoader`加载所有Jar的`META-INF/spring.factories` 
2. 根据AutoConfiguration类全限定名排序
3. 通过`@AutoConfigureOrder`排序，默认值：0
4. 通过`@AutoConfigureBefore`、`@AutoConfigureAfter`排序

参考源码:
* `org.springframework.core.io.support.SpringFactoriesLoader#loadSpringFactories(ClassLoader)`
* `org.springframework.boot.autoconfigure.AutoConfigurationSorter#getInPriorityOrder(classNames)`
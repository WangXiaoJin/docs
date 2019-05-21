## Spring Boot

* 当使用YAML格式配置日志时，你可能会遇到配置的OFF日志级别无效，因为YAML会把`OFF`/`ON`解析成`Boolean`类型的`false`/`true`。

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

* 动态修改配置

    ```
    POST http://dev.bone-cloud-zuul.banksteel.com/actuator/env
    {
        "name": "logging.level.org.springframework.security",
        "value": "debug"
    }
    ```

* 开启`tomcat accesslog`

    ```yaml
    server:
      tomcat:
        # 指定tomcat的basedir，没有指定则使用系统临时目录
        basedir: "/data/tomcat"
        accesslog:
          enabled: true
    ```
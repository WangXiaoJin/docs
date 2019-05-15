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

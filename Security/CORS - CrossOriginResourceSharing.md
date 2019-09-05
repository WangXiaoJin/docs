# Cross-Origin Resource Sharing（CORS）

## 配置 CORS

### SpringBoot集成 Cors

```yaml
spring:
  cors:
    cors-configurations:
      "[/test/**]":
        allowed-origins: "*"
        allowed-methods: "*"
        max-age: 1800
      "[/api/**]":
        allowed-origins: "*"
        allowed-methods: [GET, HEAD, POST]
        allow-credentials: true
```

```java
@Configuration
public class CorsConfig {

    private static final String CORS_CONFIG_PREFIX = "spring.cors";

    @Bean
    @ConfigurationProperties(CORS_CONFIG_PREFIX)
    public UrlBasedCorsConfigurationSource corsConfigurationSource() {
        return new UrlBasedCorsConfigurationSource();
    }

    @Bean
    public CorsFilter corsFilter(UrlBasedCorsConfigurationSource corsConfigurationSource) {
        return new CorsFilter(corsConfigurationSource);
    }

}
```

> 注：如果想控制 CorsFilter 执行顺序，则使用 FilterRegistrationBean 来配置

### SpringBoot + SpringSecurity 集成 Cors

```yaml
spring:
  cors:
    cors-configurations:
      "[/test/**]":
        allowed-origins: "*"
        allowed-methods: "*"
        max-age: 1800
      "[/api/**]":
        allowed-origins: "*"
        allowed-methods: [GET, HEAD, POST]
        allow-credentials: true
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private static final String CORS_CONFIG_PREFIX = "spring.cors";

    @Bean
    @ConfigurationProperties(CORS_CONFIG_PREFIX)
    public UrlBasedCorsConfigurationSource corsConfigurationSource() {
        return new UrlBasedCorsConfigurationSource();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatchers().anyRequest()
            ...
            // 启用 CorsFilter，自动去容器中找 corsFilter、corsConfigurationSource Bean
            .and().cors();
    }
}
```

## 参考

* [HTTP访问控制（CORS） - 【重要】](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
* [CORS 跨域 实现思路及相关解决方案](https://www.cnblogs.com/sloong/p/cors.html)
* [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
* [HTTP 请求头 `Access-Control-*` 文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Max-Age)
* [Using CORS - 跨域请求详解](https://www.html5rocks.com/en/tutorials/cors/)


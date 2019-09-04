## Spring Session

### Samples and Guides (Start Here)

##### Sample Applications that use Spring Boot

| Source | Description | Guide |
| --- | --- | --- |
| [HttpSession with Redis](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/redis) | Demonstrates how to use Spring Session to replace the HttpSession with Redis. | [HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-redis.html) |
| [HttpSession with JDBC](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/jdbc) | Demonstrates how to use Spring Session to replace the HttpSession with a relational database store. | [HttpSession with JDBC Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-jdbc.html) |
| [Find by Username](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/findbyusername) | Demonstrates how to use Spring Session to find sessions by username. | [Find by Username Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-findbyusername.html) |
| [WebSockets](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/websocket) | Demonstrates how to use Spring Session with WebSockets. | [WebSockets Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-websocket.html) |
| [WebFlux](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/webflux) | Demonstrates how to use Spring Session to replace the Spring WebFlux’s WebSession with Redis. |  |
| [HttpSession with Redis JSON serialization](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/redis-json) | Demonstrates how to use Spring Session to replace the HttpSession with Redis using JSON serialization. |  |

##### Sample Applications that use Spring Java-based configuration

| Source | Description | Guide |
| --- | --- | --- |
| [HttpSession with Redis](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/redis) | Demonstrates how to use Spring Session to replace the HttpSession with Redis. | [HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-redis.html) |
| [HttpSession with JDBC](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/jdbc) | Demonstrates how to use Spring Session to replace the HttpSession with a relational database store. | [HttpSession with JDBC Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-jdbc.html) |
| [HttpSession with Hazelcast](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/hazelcast) | Demonstrates how to use Spring Session to replace the HttpSession with Hazelcast. | [HttpSession with Hazelcast Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-hazelcast.html) |
| [Custom Cookie](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/custom-cookie) | Demonstrates how to use Spring Session and customize the cookie. | [Custom Cookie Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-custom-cookie.html) |
| [Spring Security](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/security) | Demonstrates how to use Spring Session with an existing Spring Security application. | [Spring Security Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-security.html) |
| [REST](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/rest) | Demonstrates how to use Spring Session in a REST application to support authenticating with a header. | [REST Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-rest.html) |

##### Sample Applications that use Spring XML-based configuration

| Source | Description | Guide |
| --- | --- | --- |
| [HttpSession with Redis](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/xml/redis) | Demonstrates how to use Spring Session to replace the HttpSession with a Redis store. | [HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/xml-redis.html) |
| [HttpSession with JDBC](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/xml/jdbc) | Demonstrates how to use Spring Session to replace the HttpSession with a relational database store. | [HttpSession with JDBC Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/xml-jdbc.html) |

##### Miscellaneous sample Applications

| Source | Description | Guide |
| --- | --- | --- |
| [Grails 3](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/misc/grails3) | Demonstrates how to use Spring Session with Grails 3. | [Grails 3 Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/grails3.html) |
| [Hazelcast](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/misc/hazelcast) | Demonstrates how to use Spring Session with Hazelcast in a Java EE application. | TBD |


### 总结

* 配置Spring Session，使用Jackson序列化数据

```java

import com.fasterxml.jackson.annotation.JsonTypeInfo.As;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.ObjectMapper.DefaultTyping;
import org.springframework.beans.factory.BeanClassLoaderAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.security.jackson2.CoreJackson2Module;
import org.springframework.security.jackson2.SecurityJackson2Modules;
import org.springframework.security.oauth2.client.OAuth2ClientContext;
import org.springframework.security.oauth2.client.token.AccessTokenRequest;

/**
 * 配置Spring Session，使用Jackson序列化数据，默认使用JDK序列化。
 * <p>
 * 注：OAuth2Client不可使用Jackson序列化数据。因{@link OAuth2ClientContext}为Session Scope Bean，此bean依赖{@link AccessTokenRequest} bean，
 * 而AccessTokenRequest Bean为Request Scope，其类型为JDK生成的动态代理类，在反序列化动态代理类时会报错。参考OAuth2ClientConfiguration类。
 *
 * @author WangXiaoJin
 * @date 2019-05-15 21:22
 */
@Configuration
public class SessionConfig implements BeanClassLoaderAware {

    private ClassLoader loader;

    /**
     * 定义SpringSession默认的Serializer
     *
     * @return RedisSerializer
     */
    @Bean
    public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
        return new GenericJackson2JsonRedisSerializer(objectMapper());
    }

    /**
     * 创建ObjectMapper对象，添加SpringSecurity相关的MixIn类，用于反序列化没有构造器的类。
     * <p>
     * 在最后一步开启了默认的DefaultTyping（{@code mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY)}），
     * 如果不开启此功能，{@link SecurityJackson2Modules#enableDefaultTyping(com.fasterxml.jackson.databind.ObjectMapper)}
     * 会创建{@code WhitelistTypeResolverBuilder}，只有在此白名单、配置了MixIn或明确的映射关系的类才会反序列化成功。
     * <p>
     * 如果你非常重视安全性，强烈建议你不开启默认的DefaultTyping（删除{@code mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY)}），
     * 如果有额外的POJO类需要写入Session，则自定义MixIn或映射关系（反序列规则）。这样会比较繁琐，但更安全。
     * 因为Redis的数据可能会被人恶意篡改成其他敏感类。
     *
     * @return the {@link ObjectMapper} to use
     * @see CoreJackson2Module#setupModule(com.fasterxml.jackson.databind.Module.SetupContext)
     * @see SecurityJackson2Modules#enableDefaultTyping(com.fasterxml.jackson.databind.ObjectMapper)
     */
    private ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModules(SecurityJackson2Modules.getModules(this.loader));
        // 开启默认的DefaultTyping
        mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY);
        return mapper;
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        this.loader = classLoader;
    }
}
```

### 官方文档

* [Spring Session Overview](https://spring.io/projects/spring-session)
* [官方文档](https://docs.spring.io/spring-session/docs/current/reference/html5/)
* [SpringBoot配置Spring Session](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-session)
* [HttpSession and RESTful APIs](https://docs.spring.io/spring-session/docs/current/reference/html5/#httpsession-rest)
* [Spring Security Integration - 【重要】](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security)
    * [Spring Security Remember-me Support](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security-rememberme)
    * [Spring Security Concurrent Session Control](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security-concurrent-sessions)
    * [Limitations](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security-concurrent-sessions-limitations)
* [与Redis集成相关配置及原理 - 【重要】](https://docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository)

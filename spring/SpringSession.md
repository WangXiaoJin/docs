## Spring Session

## Samples and Guides (Start Here)

### Sample Applications that use Spring Boot

| Source | Description | Guide |
| --- | --- | --- |
| [HttpSession with Redis](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/redis) | Demonstrates how to use Spring Session to replace the HttpSession with Redis. | [HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-redis.html) |
| [HttpSession with JDBC](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/jdbc) | Demonstrates how to use Spring Session to replace the HttpSession with a relational database store. | [HttpSession with JDBC Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-jdbc.html) |
| [Find by Username](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/findbyusername) | Demonstrates how to use Spring Session to find sessions by username. | [Find by Username Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-findbyusername.html) |
| [WebSockets](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/websocket) | Demonstrates how to use Spring Session with WebSockets. | [WebSockets Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-websocket.html) |
| [WebFlux](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/webflux) | Demonstrates how to use Spring Session to replace the Spring WebFlux’s WebSession with Redis. |  |
| [HttpSession with Redis JSON serialization](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/boot/redis-json) | Demonstrates how to use Spring Session to replace the HttpSession with Redis using JSON serialization. |  |

### Sample Applications that use Spring Java-based configuration

| Source | Description | Guide |
| --- | --- | --- |
| [HttpSession with Redis](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/redis) | Demonstrates how to use Spring Session to replace the HttpSession with Redis. | [HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-redis.html) |
| [HttpSession with JDBC](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/jdbc) | Demonstrates how to use Spring Session to replace the HttpSession with a relational database store. | [HttpSession with JDBC Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-jdbc.html) |
| [HttpSession with Hazelcast](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/hazelcast) | Demonstrates how to use Spring Session to replace the HttpSession with Hazelcast. | [HttpSession with Hazelcast Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-hazelcast.html) |
| [Custom Cookie](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/custom-cookie) | Demonstrates how to use Spring Session and customize the cookie. | [Custom Cookie Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-custom-cookie.html) |
| [Spring Security](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/security) | Demonstrates how to use Spring Session with an existing Spring Security application. | [Spring Security Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-security.html) |
| [REST](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/javaconfig/rest) | Demonstrates how to use Spring Session in a REST application to support authenticating with a header. | [REST Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-rest.html) |

### Sample Applications that use Spring XML-based configuration

| Source | Description | Guide |
| --- | --- | --- |
| [HttpSession with Redis](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/xml/redis) | Demonstrates how to use Spring Session to replace the HttpSession with a Redis store. | [HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/xml-redis.html) |
| [HttpSession with JDBC](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/xml/jdbc) | Demonstrates how to use Spring Session to replace the HttpSession with a relational database store. | [HttpSession with JDBC Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/xml-jdbc.html) |

### Miscellaneous sample Applications

| Source | Description | Guide |
| --- | --- | --- |
| [Grails 3](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/misc/grails3) | Demonstrates how to use Spring Session with Grails 3. | [Grails 3 Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/grails3.html) |
| [Hazelcast](https://github.com/spring-projects/spring-session/tree/2.1.5.RELEASE/samples/misc/hazelcast) | Demonstrates how to use Spring Session with Hazelcast in a Java EE application. | TBD |


## 总结

### 配置Spring Session，使用Jackson序列化数据

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

### 查看`JdkSerializationRedisSerializer`序列化至Redis的Session数据，调试使用

* 方案一 ： 代码连接Redis，获取数据

```java
/**
 * 当Redis中存储的 Session 数据是通过 {@link org.springframework.data.redis.serializer.JdkSerializationRedisSerializer}
 * 序列化时，查看相关数据非常不方便。使用此方法来打印反序列化后的数据。
 *
 * @param base64SessionId base64编码过后的SessionId，即SpringSession回写至浏览器的Cookie值
 * @param redisHost redis host
 * @param redisPort redis port
 * @param redisPwd redis password - 可选值
 * @param sessionKeyPrefix session存储至redis的key值前缀，默认为：{@code spring:session:zuul:sessions:} - 可选值
 */
public static void printJdkSerSessionInfo(String base64SessionId, String redisHost, int redisPort, String redisPwd,
    String sessionKeyPrefix) {
    byte[] bytes = Base64.getDecoder().decode(base64SessionId.getBytes(StandardCharsets.UTF_8));
    if (sessionKeyPrefix == null) {
        sessionKeyPrefix = "spring:session:zuul:sessions:";
    }
    String redisKey = sessionKeyPrefix + new String(bytes, StandardCharsets.UTF_8);

    RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
    config.setHostName(redisHost);
    if (StringUtils.isNotBlank(redisPwd)) {
        config.setPassword(RedisPassword.of(redisPwd));
    }
    config.setPort(redisPort);

    LettuceConnectionFactory connectionFactory = new LettuceConnectionFactory(config);
    connectionFactory.afterPropertiesSet();

    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setKeySerializer(new StringRedisSerializer());
    template.setHashKeySerializer(new StringRedisSerializer());
    template.setConnectionFactory(connectionFactory);
    template.afterPropertiesSet();
    HashOperations<Object, Object, Object> hash = template.opsForHash();

    // 获取Session属性值
    Object securityContext = hash.get(redisKey, "sessionAttr:SPRING_SECURITY_CONTEXT");
    Object maxInactiveInterval = hash.get(redisKey, "maxInactiveInterval");
    ZonedDateTime creationTime = Instant.ofEpochMilli(((Long) hash.get(redisKey, "creationTime")))
        .atZone(ZoneId.systemDefault());
    ZonedDateTime lastAccessedTime = Instant.ofEpochMilli(((Long) hash.get(redisKey, "lastAccessedTime")))
        .atZone(ZoneId.systemDefault());

    // 打印信息
    System.out.println("【securityContext】: " + securityContext);
    System.out.println("【maxInactiveInterval】: " + maxInactiveInterval);
    System.out.println("【creationTime】: " + creationTime);
    System.out.println("【lastAccessedTime】: " + lastAccessedTime);

    // 销毁连接
    connectionFactory.destroy();

}
```

* 方案二 ：不需要Redis依赖包，简单方便

```javascript
// 需要反序列化的数据，直接从Redis中拷贝
var val = "\xAC\xED\x00\x05sr\x00\x11java.lang.Integer\x12\xE2\xA0\xA4\xF7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xAC\x95\x1D\x0B\x94\xE0\x8B\x02\x00\x00xp\x00\x00\x07\x08";
// 转义成 Java 需要的 byte 数组
var rawVal = new Int8Array([...val].map(code => code.charCodeAt(0))).toString();
console.log(rawVal);
```

```java
// bytes 的值为上一步的 JS操作返回的值
byte[] bytes = {-84,-19,0,5,115,114,0,17,106,97,118,97,46,108,97,110,103,46,73,110,116,101,103,101,114,18,-30,-96,-92,-9,-127,-121,56,2,0,1,73,0,5,118,97,108,117,101,120,114,0,16,106,97,118,97,46,108,97,110,103,46,78,117,109,98,101,114,-122,-84,-107,29,11,-108,-32,-117,2,0,0,120,112,0,0,7,8};

// 使用 apache-commons 包工具类
Object obj = SerializationUtils.deserialize(bytes);
System.out.println(obj);

// 使用 spring-core 包工具类
System.out.println(new DeserializingConverter().convert(bytes));
```

## 官方文档

* [Spring Session Overview](https://spring.io/projects/spring-session)
* [官方文档](https://docs.spring.io/spring-session/docs/current/reference/html5/)
* [SpringBoot配置Spring Session](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-session)
* [HttpSession and RESTful APIs](https://docs.spring.io/spring-session/docs/current/reference/html5/#httpsession-rest)
* [Spring Security Integration - 【重要】](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security)
    * [Spring Security Remember-me Support](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security-rememberme)
    * [Spring Security Concurrent Session Control](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security-concurrent-sessions)
    * [Limitations](https://docs.spring.io/spring-session/docs/current/reference/html5/#spring-security-concurrent-sessions-limitations)
* [与Redis集成相关配置及原理 - 【重要】](https://docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository)

## Spring Data Redis

### 总结

* **org.springframework.data.redis.support** 包下提供了`atomic counters`、`JDK Collections`等组件，
[Support Classes](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#redis:support)。

* By default, all `LettuceConnection` instances created by the `LettuceConnectionFactory` share the same thread-safe
 `native connection` for all non-blocking and non-transactional operations. To use a dedicated connection each time, 
 set `shareNativeConnection` to `false`. `LettuceConnectionFactory` can also be configured to use a `LettucePool` for pooling 
 blocking and transactional connections or all connections if `shareNativeConnection` is set to `false`.

* For cases where you need a certain template view, declare the view as a dependency and inject the template. 
The container automatically performs the conversion, eliminating the `opsFor[X]` calls, as shown in the following example:
    
    ```java
    public class Example {
    
      // inject the actual template
      @Autowired
      private RedisTemplate<String, String> template;
    
      // inject the template as ListOperations
      @Resource(name="redisTemplate")
      private ListOperations<String, String> listOps;
    
      public void addLink(String userId, URL url) {
        listOps.leftPush(userId, url.toExternalForm());
      }
    }
    ```
    
    > [官方文档](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#redis:template)

* Write to Master, Read from Replica - [官方文档](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#redis:write-to-master-read-from-replica)

    ```java
    @Configuration
    class WriteToMasterReadFromReplicaConfiguration {
    
      @Bean
      public LettuceConnectionFactory redisConnectionFactory() {
    
        LettuceClientConfiguration clientConfig = LettuceClientConfiguration.builder()
          .readFrom(SLAVE_PREFERRED)
          .build();
    
        RedisStandaloneConfiguration serverConfig = new RedisStandaloneConfiguration("server", 6379);
    
        return new LettuceConnectionFactory(serverConfig, clientConfig);
      }
    }
    ```

* Redis Sentinel Support - [官方文档](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#redis:sentinel)

    ```java
    /**
     * Jedis
     */
    @Bean
    public RedisConnectionFactory jedisConnectionFactory() {
      RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
      .master("mymaster")
      .sentinel("127.0.0.1", 26379)
      .sentinel("127.0.0.1", 26380);
      return new JedisConnectionFactory(sentinelConfig);
    }
    
    /**
     * Lettuce
     */
    @Bean
    public RedisConnectionFactory lettuceConnectionFactory() {
      RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
      .master("mymaster")
      .sentinel("127.0.0.1", 26379)
      .sentinel("127.0.0.1", 26380);
      return new LettuceConnectionFactory(sentinelConfig);
    }
    ```

### 文档

* [SpringDataRedis 官方文档](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/)
* [Redis Transactions](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#tx)
* [Pipelining](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#pipeline)
* [Redis Scripting](https://docs.spring.io/spring-data/redis/docs/2.1.6.RELEASE/reference/html/#scripting)
* [Lettuce 官方文档](https://lettuce.io/core/release/reference/index.html)

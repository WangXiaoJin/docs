## Jackson

* [Jackson JSON Tutorial](https://www.baeldung.com/jackson)
* [Jackson 框架的高阶应用](https://www.ibm.com/developerworks/cn/java/jackson-advanced-application/index.html)
* [Jackson ObjectMapper](http://tutorials.jenkov.com/java-json/jackson-objectmapper.html)
* [Jackson JsonNode](http://tutorials.jenkov.com/java-json/jackson-jsonnode.html)
* [Jackson JsonParser](http://tutorials.jenkov.com/java-json/jackson-jsonparser.html)
* [Jackson JsonGenerator](http://tutorials.jenkov.com/java-json/jackson-jsongenerator.html)
* [Jackson Annotations](http://tutorials.jenkov.com/java-json/jackson-annotations.html)

### 总结

* 使用Jackson序列化对象时，如果对象的某个属性使用了JDK动态代理（属性的实际类型为代理类），
则使用Jackson反序列化时会抛异常。此时可使用以下方案解决（针对动态代理的属性类型使用JDK序列化）：

```java
import com.easycodebox.spring.cloud.oauth2.AccessTokenRequestMixin.AccessTokenRequestDeserializer;
import com.easycodebox.spring.cloud.oauth2.AccessTokenRequestMixin.AccessTokenRequestSerializer;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.annotation.JsonTypeInfo.Id;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import java.io.IOException;
import java.util.Map;
import org.springframework.security.oauth2.client.token.AccessTokenRequest;
import org.springframework.security.oauth2.common.util.SerializationUtils;

/**
 * 当启用{@link org.springframework.security.oauth2.config.annotation.web.configuration.EnableOAuth2Client}时，
 * 会初始化{@link org.springframework.security.oauth2.config.annotation.web.configuration.OAuth2ClientConfiguration#accessTokenRequest(Map, String)} Bean，
 * 而此{@code AccessTokenRequest} Bean为 request scope bean，此bean为代理bean，所以通过Jackson反序列化时会失败。
 *
 * @author WangXiaoJin
 * @date 2019-05-31 16:04
 */
@JsonTypeInfo(use = Id.NONE)
@JsonSerialize(using = AccessTokenRequestSerializer.class)
@JsonDeserialize(using = AccessTokenRequestDeserializer.class)
public class AccessTokenRequestMixin {

    public static class AccessTokenRequestDeserializer extends JsonDeserializer<AccessTokenRequest> {

        @Override
        public AccessTokenRequest deserialize(JsonParser p, DeserializationContext des)
            throws IOException {
            return SerializationUtils.deserialize(p.getBinaryValue());
        }
    }

    public static class AccessTokenRequestSerializer extends JsonSerializer<AccessTokenRequest> {

        @Override
        public void serialize(AccessTokenRequest value, JsonGenerator gen, SerializerProvider serializers)
            throws IOException {
            gen.writeBinary(SerializationUtils.serialize(value));
        }
    }
}
```

```java
ObjectMapper mapper = new ObjectMapper();
//启用自定义的Mixin
mapper.addMixIn(AccessTokenRequest.class, AccessTokenRequestMixin.class);
// 开启默认的DefaultTyping
mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY);
```

* 自定义第三方框架的 序列化/反序列化 规则：参考 `spring-security-core`的`org.springframework.security.jackson2.CoreJackson2Module`类
# Dubbo泛化调用

## 示例

```java
import com.alibaba.dubbo.common.utils.PojoUtils;
import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.ConsumerConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import com.alibaba.dubbo.config.spring.ReferenceBean;
import com.alibaba.dubbo.config.utils.ReferenceConfigCache;
import com.alibaba.dubbo.rpc.service.GenericService;

public class Test {

    public static void main(String[] args) {
        //创建 ApplicationConfig
        ApplicationConfig app = new ApplicationConfig();
        // 应用名可自行设定
        app.setName("app-test");

        //创建注册中心配置
        RegistryConfig registry = new RegistryConfig();
        registry.setAddress("zookeeper://dubbo1:2181?backup=dubbo2:2181,dubbo3:2181");
        // 可选配置
        registry.setClient("curator");

        //创建注册中心配置
        ConsumerConfig consumer = new ConsumerConfig();
        consumer.setCheck(false);
        consumer.setRetries(0);

        // 创建 ReferenceBean
        ReferenceBean<GenericService> bean = new ReferenceBean<>();
        bean.setApplication(app);
        bean.setRegistry(registry);
        bean.setConsumer(consumer);
        // 调用指定服务 - 指定URL后 registry 可以不用配置
        //bean.setUrl("dubbo://192.168.254.104:23000");
        // API接口类名
        bean.setInterface("com.xxx.DubboApi");
        // 启用泛化接口调用
        bean.setGeneric("true");
        // 初始化 Bean
        GenericService service = bean.get();
        try {
            // 参数类型 - 如想自动生成类型字符串，使用：ReflectUtils.getName()
            String[] paramTypes = new String[]{
                "java.lang.String",
                "com.xxx.dto.XxxDTO"
            };

            // 参数值
            Object[] params = new Object[]{
                "7f30bdee",
                // POJO 转 HashMap
                PojoUtils.generalize(new XxxDTO()) 
            };
            // 调用API接口的 use 方法
            Object result = service.$invoke("use", paramTypes, params);
            System.out.println(result);
        } finally {
            // 销毁资源
            bean.destroy();
        }
    }
}
```

* 泛化调用可以用HashMap代替Pojo传参，只需Map中增加一个`class`属性指明Pojo类型，便于反序列化使用
* `GenericService`对象创建较重，建议使用缓存。可参考`ReferenceConfigCache`类（`ReferenceConfigCache.getCache().get(genericService)`)，
但其存在缓存无限制的弊端。


## 文档

* [泛化调用（客户端泛化）](https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/advanced-features-and-usage/service/generic-reference/)



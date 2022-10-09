# Servlet-常用

## 文件上传

* tomcat在处理`javax.servlet.http.HttpServletRequest`参数之前会执行`org.apache.catalina.connector.Request.parseParameters()`，
此方法里面包含了解析`multipart/form-data`逻辑，此逻辑会把请求体中的文件数据存到临时目录中，并把相关信息保存至`javax.servlet.http.Part`

* SpringMVC的`spring.servlet.multipart.resolve-lazily: true`会让SpringMVC延迟上述`javax.servlet.http.Part`解析，
但如果在在此之前调用了HttpServletRequest获取参数还是会提前触发Part解析。参考源码：`org.springframework.web.servlet.DispatcherServlet.checkMultipart()`

## Request/Response Stream 可重复读

参考`logback-access`的`TeeFilter`/`TeeHttpServletRequest`/`TeeServletOutputStream`

## SpringBoot注册Filter、Servlet、Listener机制

SpringBoot支持自动注册下面的Bean至Servlet容器中：

* Servlet
* Filter
* ServletRegistrationBean
* FilterRegistrationBean
* DelegatingFilterProxyRegistrationBean
* ServletListenerRegistrationBean
* ServletContextInitializer
* ServletContextAttributeListener
* ServletRequestListener
* ServletRequestAttributeListener
* HttpSessionAttributeListener
* HttpSessionListener
* ServletContextListener

源码路线图：
1. 获取上面可注册Bean及排序规则参考`ServletContextInitializerBeans`
2. `org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.getServletContextInitializerBeans()`
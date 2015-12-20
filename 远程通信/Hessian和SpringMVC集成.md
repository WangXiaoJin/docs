## Hessian和SpringMVC集成

#####　`【原创】` :heart_eyes:
---

如果你还在为WebService的性能、配置等烦恼时，不妨试试Hessian，配置就是so 简单

###服务端代码

1. pom配置：

	```xml
	<dependency>
		<groupId>com.caucho</groupId>
		<artifactId>hessian</artifactId>
		<version>4.0.38</version>
	</dependency>
	```

2. ws-server.xml：

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd"
		default-lazy-init="true">
	
		<bean name="/ws/user" class="org.springframework.remoting.caucho.HessianServiceExporter">
			<property name="service" ref="UserWsServer" />
			<property name="serviceInterface" value="com.xx.ws.UserWsService" />
		</bean>
		
	</beans>
	```

3. UserWsService 接口：

	```java
	public interface UserWsService {
		
		/**
		 * ErrorContext为自定义异常类。因为此为通信功能，最好自己实现一个异常类，异常类中包含错误
		 * 的code、msg等信息
		 */
		String hello(String msg) throws ErrorContext;
		
	}
	```

4. UserWsServiceImpl 实现类：

	```java
	@Service("UserWsServer")
	public class UserWsServiceImpl implements UserWsService {
		
		/**
		 * ErrorContext为自定义异常类。因为此为通信功能，最好自己实现一个异常类，异常类中包含错误
		 * 的code、msg等信息
		 */
		@Override
		public String hello(String msg) throws ErrorContext {
			return msg + "：hello";
		}
		
	}
	```

5. web.xml - 只列出相关的配置，其他配置省略：

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
		xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
		version="2.5">
		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
				classpath:applicationContext.xml
	        </param-value>
		</context-param>
		
		<!-- 其他配置省略......... -->
		
		<servlet>
			<servlet-name>SpringMVC</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>
					classpath*:mvc-*.xml,
					classpath:ws-server.xml
				</param-value>
			</init-param>
		</servlet>
		<servlet-mapping>
			<servlet-name>SpringMVC</servlet-name>
			<url-pattern>/</url-pattern>
		</servlet-mapping>
	
		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>
	
	</web-app>
	```

---

###客户端代码

1. pom配置：

	```xml
	<dependency>
		<groupId>com.caucho</groupId>
		<artifactId>hessian</artifactId>
		<version>4.0.38</version>
	</dependency>
	```

2. ws-client.xml
	
	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd"
		default-lazy-init="true">
	
		<bean id="userWsService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
			<property name="serviceUrl" value="http://localhost:8080/xx/ws/user" />
			<property name="serviceInterface" value="com.xx.ws.UserWsService" />
		</bean> 
		
	</beans>
	```

3. UserWsService 接口：

	```java
	public interface UserWsService {
		
		/**
		 * ErrorContext为自定义异常类。因为此为通信功能，最好自己实现一个异常类，异常类中包含错误
		 * 的code、msg等信息
		 */
		String hello(String msg) throws ErrorContext;
		
	}
	```

5. web.xml - 只列出相关的配置，其他配置省略：

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
		xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
		version="2.5">
		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
				classpath:applicationContext.xml,
				classpath:ws-client.xml,
	        </param-value>
		</context-param>
		
		<!-- 其他配置省略......... -->
	
	</web-app>
	```

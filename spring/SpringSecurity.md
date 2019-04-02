## Spring Security

### Spring Security

* [架构及实现 - 【重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#core-components)
  
  * SecurityContextHolder, SecurityContext and Authentication Objects
  * UserDetailsService
  * GrantedAuthority
  * Authentication
  * ExceptionTranslationFilter
  * AuthenticationEntryPoint
  * Authentication Mechanism
  * Storing the SecurityContext between requests
  * Access-Control (Authorization) in Spring Security
  * Localization
  * The AuthenticationManager, ProviderManager and AuthenticationProvider
  * Erasing Credentials on Successful Authentication
  * Password Encoding
  
  提供了三种`Authentication`数据共享模式：
  * `SecurityContextHolder.MODE_GLOBAL`
  * `SecurityContextHolder.MODE_THREADLOCAL`
  * `SecurityContextHolder.MODE_INHERITABLETHREADLOCAL`

  > 注：另一种常用实现方法 [Concurrency Support - 让子线程使用SecurityContext对象 -【重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#concurrency)

* [What does "ROLE_" mean and why do I need it on my role names](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#appendix-faq-role-prefix)
* [How do I define the secured URLs within an application dynamically](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#appendix-faq-dynamic-url-metadata)
* [角色继承 - 【重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#authz-hierarchical-roles)

  ```xml
  <bean id="roleVoter" class="org.springframework.security.access.vote.RoleHierarchyVoter">
      <constructor-arg ref="roleHierarchy" />
  </bean>
  <bean id="roleHierarchy"
          class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
      <property name="hierarchy">
          <value>
              ROLE_ADMIN > ROLE_STAFF
              ROLE_STAFF > ROLE_USER
              ROLE_USER > ROLE_GUEST
          </value>
      </property>
  </bean>
  ```

* [Expression-Based Access Control - Spirng EL表达式权限控制 - 【非常重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#el-access)

* [Filter配置顺序 - 【重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#filter-ordering)
* [Filter执行顺序及自定义Filter](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#ns-custom-filters)
* [Spring Security集成单元测试 - 【重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#test)
* [Spring Security相关所有demo - 【重要】](https://github.com/spring-projects/spring-security/tree/master/samples)
/ [spring-security-oauth demo](https://github.com/spring-projects/spring-security-oauth/tree/master/samples)

* [JSP Tag Libraries - 重要](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#taglibs)
* [Spring MVC Integration - 集成mvcMatchers/@AuthenticationPrincipal/CsrfToken/Async至SpringMVC -【重要】](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#mvc)
  
  > 注: To enable Spring Security integration with Spring MVC add the `@EnableWebSecurity` annotation to your configuration.
  Spring Security provides the configuration using Spring MVC’s `WebMvcConfigurer`. This means that if you are using more 
  advanced options, like integrating with `WebMvcConfigurationSupport` directly, then you will need to 
  manually provide the Spring Security configuration.
  
  > 注：It is important to keep the `CsrfToken` a secret from other domains. This means if you are 
  using `Cross Origin Sharing (CORS)`, you **should NOT** expose the CsrfToken to any external domains.

* [Run-As Authentication Replacement](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#runas)
* [Remember-Me Authentication](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#remember-me)

* [HttpFirewall - 拦截恶意请求](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#request-matching)
  
  > If you are using `new MockHttpServletRequest()` it currently creates an HTTP method as an `empty String ""`. 
  This is an invalid HTTP method and will be **rejected** by Spring Security. You can resolve this by replacing 
  it with `new MockHttpServletRequest("GET", "")`. See [SPR_16851](https://jira.spring.io/browse/SPR-16851) for an issue requesting to improve this.

* [CommonOAuth2Provider - 提供了Google/GitHub/Facebook/Okta通用配置，简化配置](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#oauth2login-common-oauth2-provider)
* [Overriding Spring Boot 2.x Auto-configuration - OAuth2login](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#oauth2login-override-boot-autoconfig)
* [配置JwtDecoder RestOperations的超时时间 - OAuth 2.0 Resource Server](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#oauth2resourceserver-timeouts)
* [OAuth 2.0 Login — Advanced Configuration](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#oauth2login-advanced)
* [配置Authentication认证数据源（内存、数据库、LDAP）](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#jc-authentication)
* [SavedRequest s and the RequestCache Interface](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#request-caching)
* [Multiple HttpSecurity](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#multiple-httpsecurity)
  
  ```java
  @EnableWebSecurity
  public class MultiHttpSecurityConfig {
      @Bean                                                             1
      public UserDetailsService userDetailsService() throws Exception {
          // ensure the passwords are encoded properly
          UserBuilder users = User.withDefaultPasswordEncoder();
          InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
          manager.createUser(users.username("user").password("password").roles("USER").build());
          manager.createUser(users.username("admin").password("password").roles("USER","ADMIN").build());
          return manager;
      }
  
      @Configuration
      @Order(1)                                                        2
      public static class ApiWebSecurityConfigurationAdapter extends WebSecurityConfigurerAdapter {
          protected void configure(HttpSecurity http) throws Exception {
              http
                  .antMatcher("/api/**")                               3
                  .authorizeRequests()
                      .anyRequest().hasRole("ADMIN")
                      .and()
                  .httpBasic();
          }
      }
  
      @Configuration                                                   4
      public static class FormLoginWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {
  
          @Override
          protected void configure(HttpSecurity http) throws Exception {
              http
                  .authorizeRequests()
                      .anyRequest().authenticated()
                      .and()
                  .formLogin();
          }
      }
  }
  ```

* [Post Processing Configured Objects - 修改或替换某些实例](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#post-processing-configured-objects)
  
   ```java
  @Override
  protected void configure(HttpSecurity http) throws Exception {
      http
          .authorizeRequests()
              .anyRequest().authenticated()
              .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                  public <O extends FilterSecurityInterceptor> O postProcess(
                          O fsi) {
                      fsi.setPublishAuthorizationSuccess(true);
                      return fsi;
                  }
              });
  }
  ```

* [Custom DSLs](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#jc-custom-dsls)
* [Method Security](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#jc-method)

* [Spring Security Dependencies - 查看Spring Security相关包及依赖关系](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#appendix-dependencies)

* [Design of the Namespace](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#design-of-the-namespace)

  * **Web/HTTP Security** - the most complex part. Sets up the filters and related service beans used to apply the framework authentication mechanisms, to secure URLs, render login and error pages and much more.
  * **Business Object (Method) Security** - options for securing the service layer.
  * **AuthenticationManager** - handles authentication requests from other parts of the framework.
  * **AccessDecisionManager** - provides access decisions for web and method security. A default one will be registered, but you can also choose to use a custom one, declared using normal Spring bean syntax.
  * **AuthenticationProviders** - mechanisms against which the authentication manager authenticates users. The namespace provides supports for several standard options and also a means of adding custom beans declared using a traditional syntax.
  * **UserDetailsService** - closely related to authentication providers, but often also required by other beans.

  You can use multiple <intercept-url> elements to define different access requirements for different sets of URLs, 
  but they will be evaluated in the order listed and the first match will be used. So you must put the most specific 
  matches at the top. You can also add a method attribute to limit the match to a particular HTTP method (GET, POST, PUT etc.).

  ```xml
  <beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd">
  
    <http>
      <intercept-url pattern="/dbAdmin/**" access="hasRole('DB_ADMIN')" method="GET"/>
      <intercept-url pattern="/**" access="hasRole('USER')" method="GET"/>
      <form-login/>
      <logout/>
    </http>
  
    <authentication-manager>
      <authentication-provider>
        <user-service>
          <!-- Password is prefixed with {noop} to indicate to DelegatingPasswordEncoder that
          NoOpPasswordEncoder should be used. This is not safe for production, but makes reading
          in samples easier. Normally passwords should be hashed using BCrypt -->
          <user name="jimi" password="{noop}jimispassword" authorities="ROLE_USER, ROLE_ADMIN"/>
          <user name="bob" password="{noop}bobspassword" authorities="ROLE_USER"/>
        </user-service>
      </authentication-provider>
    </authentication-manager>
  
  </beans:beans>
  ```
  
  You can use multiple authentication-provider elements, in which case the providers will be queried in the order they are declared.

  * [Adding HTTP/HTTPS Channel Security](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#ns-requires-channel)
  * [Session Management](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#ns-session-mgmt)
    
    > 通过`<logout delete-cookies="JSESSIONID" />`解决，登出后直接登录出现的问题
  
  * [Concurrent Session Control - 限制同一用户同时登录数](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#ns-concurrent-sessions)
  * [Session Fixation Attack Protection](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#ns-session-fixation)
  * [OpenID Support](https://docs.spring.io/spring-security/site/docs/5.1.4.RELEASE/reference/htmlsingle/#ns-openid)
  
> You can use the attribute `filters = "none"` as an alternative to supplying a filter bean list. 
This will `omit` the request pattern from the security filter chain entirely. Note that anything matching 
this path will then have no authentication or authorization services applied and will be freely accessible.
 If you want to make use of the contents of the `SecurityContext` contents during a request, 
 then it must have passed through the security filter chain. Otherwise the `SecurityContextHolder`
 will not have been populated and the contents will be `null`.
 
 
### Spring Security Oauth

* [spring-security-oauth schema.sql参考](https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql)

* [OAuth 2 Developers Guide - `Authorization Server`/`Resource Server`/`OAuth 2.0 Client`配置及使用](https://projects.spring.io/spring-security-oauth/docs/oauth2.html)

* [How to log out when using JWT - 使用redis存储已登出的token](https://dev.to/_arpy/how-to-log-out-when-using-jwt-4ajm)

* [Spring Boot and OAuth2 - 【重要】](https://spring.io/guides/tutorials/spring-boot-oauth2/)

* [OAuth2 Autoconfig - 自动配置`Authorization/Resource Server` - 【重要】](https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/htmlsingle/)

### Spring Cloud Security

* [官方文档](https://spring.io/projects/spring-cloud-security#overview)

### **总结**

* `spring-security 5.*`版本下包含了oauth2的子项目，此子项目是以后重点维护项目。将从`org.springframework.security.oauth:spring-security-oauth2`
项目慢慢转移至`spring-security 5.*`。`spring-security 5.*`中没有提供`Authorization Server`服务，只有`spring-security-oauth2`提供了`Authorization Server`服务。
`spring-security:5.1.3`提供了`spring-security-oauth2-resource-server`/`spring-security-oauth2-client`。
  
  > 注：[OAuth 2.0 各项目支持的特性及常见问题](https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Features-Matrix)
  
* 提供`UserInfoRestTemplateCustomizer` 或者`UserInfoTemplateFactory`控制`OAuth2RestTemplate`功能。参考：`ResourceServerTokenServicesConfiguration.userInfoRestTemplateFactory()`

  ```java
  @Bean
  public UserInfoRestTemplateCustomizer customHeader() {
    return restTemplate ->
        restTemplate.getInterceptors().add(new MyCustomInterceptor());
  }
  ```
  
  或者自己创建`OAuth2RestTemplate`：
  ```java
  @Bean
  public OAuth2RestTemplate oauth2RestTemplate(OAuth2ClientContext oauth2ClientContext,
          OAuth2ProtectedResourceDetails details) {
      return new OAuth2RestTemplate(details, oauth2ClientContext);
  }
  ```

* 记不清OAuth2有哪些grant_type？
  
  查看`BaseOAuth2ProtectedResourceDetails`所有实现类调用`setGrantType()`方法传入的值，包含：
  * `client_credentials`
  * `authorization_code`
  * `implicit`
  * `password`
  * `refresh_token`（附带）

* spring-security-oauth2中所有`endpoint`定义都在`org.springframework.security.oauth2.provider.endpoint`包下，方便追踪每个endpoint的具体实现。
如：`AuthorizationEndpoint`。

* `org.springframework.security.oauth2.common.util.OAuth2Utils`查看OAuth2传递的参数名

* `WebSecurityConfigurerAdapter`类上配置`@Order(100)`，当有多个WebSecurityConfigurerAdapter Bean Order相同时启动会报错。
当容器中有多个WebSecurityConfigurerAdapter Bean时，请求会依次按照Order从低到高进行匹配，匹配成功则使用此WebSecurityConfigurerAdapter，
不会按照请求路径最大化匹配的。

* `org.springframework.security.config.annotation.web.builders.FilterComparator`类中设置了所有内置Filter的Order。

* `WebSecurity`是SpringSecurity核心类。`WebSecurity`初始化在`org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration.setFilterChainProxySecurityConfigurer`
方法中初始化，所有的`SecurityConfigurer`对象都是在此方法中配置到`WebSecurity`实例中。
  
* 默认的Filter是在`WebSecurityConfigurerAdapter.init()`方法中配置的，便于基础功能实现。很多类的配置都是在`WebSecurityConfigurerAdapter`中
完成的。`SecurityConfigurer`类的`init`方法在`webSecurity.build()`方法中执行，而`webSecurity.build()`通过`WebSecurityConfiguration.springSecurityFilterChain()`方法调用。

* `FilterChainProxy`类是SpringSecurity处理请求的Filter链路，用于决定当前请求会被什么Filter拦截。

#### OAuth Server

* 如果你没有提供`AuthorizationServerConfigurer`实例时，`spring-security-oauth2-autoconfigure`默认给你创建
`OAuth2AuthorizationServerConfiguration` Bean，此bean负责解析`security.oauth2.client`配置项，创建InMemoryClientDetailsService及NoOpPasswordEncoder。
如果你想使用JdbcClientDetailsService或自定义PasswordEncoder时，需要自己提供`AuthorizationServerConfigurer`实例。

* 有关`OAuth2 Authorization Server`相关接口的用户认证使用`ClientDetailsService`(参考`AuthorizationServerSecurityConfigurer.init()`)，其他的则用`UserDetailsService`。

#### OAuth Resource

* `jwt.keyUri`/`jwk.keySetUri`/`userInfoUri`/`tokenInfoUri`四种验证及获取用户信息方式，参考`ResourceServerProperties.doValidate()`方法查看验证规则。
  
  * `tokenInfoUri` - 访问Server`/check_token`接口返回access token对象,。如果服务返回400，说明传入token参数无效。
  * `userInfoUri` - 用户信息接口
  * `jwtkeyUri` - 启动应用时调用此URL加载公钥，只返回一个
  * `jwkkeySetUri` - 不需要在应用启动时去获取所有的公钥。当需要验证AccessToken时，从中解析出`kid`，调用`JwkDefinitionSource.getDefinitionLoadIfNecessary(keyId)`返回具体的公钥信息。如果本地没有，则此时发送请求获取所有的公钥信息。
  
  > 注：`spring-security-oauth2:2.3.5.RELEASE` `Authorization Server` 默认不支持`JWKs`，如果你想`Authorization Server`支持`JWKs`，需要自行配置`所有的私/公钥对`及`JWK Set URI endpoint`。
  参考官方文档[Is Authorization Server Compatible with Spring Security 5.1 Resource Server and Client?](https://docs.spring.io/spring-security-oauth2-boot/docs/current-SNAPSHOT/reference/htmlsingle/#oauth2-boot-authorization-server-spring-security-oauth2-resource-server)
  
  > 注：[参考文档](https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/htmlsingle/#boot-features-security-oauth2-resource-server)
  
  > `OAuth2ClientAuthenticationProcessingFilter.attemptAuthentication()`方法中`OAuth2Authentication result = tokenServices.loadAuthentication(accessToken.getValue())`用到了相关逻辑

### 非官方参考文档

* [Spring cloud oauth2.0学习总结](https://blog.csdn.net/j754379117/article/details/70175198)
* [OAuth2 源码分析(一.核心类）](https://blog.csdn.net/qq_30905661/article/details/81112305)
* [Simple Single Sign-On with Spring Security OAuth2](https://www.baeldung.com/sso-spring-security-oauth2)
* [Spring Security 5 – OAuth2 Login](https://www.baeldung.com/spring-security-5-oauth2-login)
* [OAuth2 – @EnableResourceServer vs @EnableOAuth2Sso](https://www.baeldung.com/spring-security-oauth2-enable-resource-server-vs-enable-oauth2-sso)

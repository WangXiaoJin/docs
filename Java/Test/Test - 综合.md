# Test - 综合


## 术语

    * [Mock Objects](https://en.wikipedia.org/wiki/Mock_Object): Article in Wikipedia.

    * [MockObjects.com](http://www.mockobjects.com/): Web site dedicated to mock objects, a technique for improving the design of code within test-driven development.

## 框架及组件

* 单元测试框架

    * [JUnit](https://www.junit.org/): “A programmer-friendly testing framework for Java”. Used by the Spring Framework in its test suite
    and supported in the [Spring TestContext Framework](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-framework).

    * [TestNG](https://testng.org/): A testing framework inspired by JUnit with added support for test groups, data-driven testing, distributed testing,
    and other features. Supported in the [Spring TestContext Framework](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-framework)

    * [Mockito](https://mockito.github.io/):【推荐】Java mock library based on the [Test Spy](http://xunitpatterns.com/Test%20Spy.html) pattern. Used by the Spring Framework in its test suite.
        * [Github Wiki](https://github.com/mockito/mockito/wiki)
        * [Docs](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)

    * [PowerMock](http://powermock.github.io/):【推荐】扩展了其他Mock框架。 PowerMock uses a custom classloader and bytecode manipulation
    to enable mocking of `static methods`, `constructors`, `final classes and methods`, `private methods`, `removal of static initializers` and more.
     Currently PowerMock supports `EasyMock` and `Mockito`.

    * [EasyMock](https://easymock.org/): Java library “that provides Mock Objects for interfaces (and objects through the class extension) by
    generating them on the fly using Java’s proxy mechanism.”

    * [JMock](https://jmock.org/): Library that supports test-driven development of Java code with mock objects.

    * [SpringMockK](https://github.com/Ninja-Squad/springmockk): Support for Spring Boot integration tests written in Kotlin using [MockK](https://mockk.io/) instead of Mockito.


* 独立组件

    * [SpringBoot - Testing](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing)

    * [Spring - Testing](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testing)
        * `Client-Side REST Tests` - `MockRestServiceServer`

    * [spring-security-test](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/#test)

    * [Spring Cloud Contract](https://spring.io/projects/spring-cloud-contract)
        * [Github地址](https://github.com/spring-cloud/spring-cloud-contract/)

    * [Spring - Embedded Database Support](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/data-access.html#jdbc-embedded-database-support)

    * [spring-amqp - Testing](https://docs.spring.io/spring-amqp/docs/2.2.6.RELEASE/reference/html/#testing)

    * [spring-cloud-stream - Testing](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/2.1.3.RELEASE/multi/multi__testing.html)

    * [WireMock](http://wiremock.org/) - 【推荐】Mock your APIs for fast, robust and comprehensive testing.
        WireMock is a simulator for HTTP-based APIs. Some might consider it a `service virtualization` tool or a `mock server`.
        * 推荐链接
            * [Stubbing](http://wiremock.org/docs/stubbing/)
            * [Verifying](http://wiremock.org/docs/verifying/)
            * [Request Matching](http://wiremock.org/docs/request-matching/)
            * [Record and Playback (New)](http://wiremock.org/docs/record-playback/)
            * [Record and Playback (Legacy)](http://wiremock.org/docs/record-playback-legacy/)
            * [Response Templating](http://wiremock.org/docs/response-templating/)
            * [Simulating Faults](http://wiremock.org/docs/simulating-faults/)
            * [Extending WireMock](http://wiremock.org/docs/extending-wiremock/)
            * [Stub Metadata](http://wiremock.org/docs/stub-metadata/)
            * [Admin API Reference](http://wiremock.org/docs/api/)
            * [示例](https://github.com/tomakehurst/wiremock/blob/master/src/test/java/ignored/Examples.java#374)
        * [MockLab](http://get.mocklab.io/) - API在线文档，可以Mock HTTP请求。
            Ship better apps earlier by mocking the APIs you depend on.
            Rapidly simulate APIs for faster parallel development and more comprehensive testing.
            MockLab is a hosted API simulator built on WireMock, with an intuitive web UI, team collaboration and nothing to install.
            The 100% compatible API supports drop-in replacement of the WireMock server with a single line of code.

    * [OkHttp MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) - 功能单一

    * [Curator-Test](https://github.com/apache/curator/tree/master/curator-test) - 本地启用ZooKeeper Server

    * [embedded-redis](https://github.com/ozimov/embedded-redis) - 本地启用Redis Server

    * [AssertJ](https://joel-costigliola.github.io/assertj/): “Fluent assertions for Java”, including support for Java 8 lambdas, streams, and other features.

    * [DbUnit](https://www.dbunit.org/): JUnit extension (also usable with Ant and Maven) that is targeted at database-driven projects and,
    among other things, puts your database into a known state between test runs.

    * [Testcontainers](https://www.testcontainers.org/): Java library that supports JUnit tests, providing lightweight, throwaway instances
    of common databases, Selenium web browsers, or anything else that can run in a Docker container.

    * [The Grinder](https://sourceforge.net/projects/grinder/): Java load testing framework.

    * `feign-mock` - 用于测试`OpenFeign`
        * [官网](https://github.com/OpenFeign/feign/tree/master/mock)
        * [示例](https://github.com/OpenFeign/feign/blob/master/mock/src/test/java/feign/mock/MockClientTest.java)
        * `feign-mock`的替代方案：[How to test with Spring Boot, Feign Client and WireMock](http://www.thecodingpark.com/how-to-test-with-spring-boot-feign-client-and-wiremock/)

## 文档

* [Faking OAuth2 Single Sign-on in Spring, 3 Ways](https://engineering.pivotal.io/post/faking_oauth_sso/)

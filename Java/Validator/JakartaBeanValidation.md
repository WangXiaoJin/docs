# Jakarta Bean Validation

## Jakarta Bean Validation

### Changes between Jakarta Bean Validation 2.0 and Bean Validation 2.0

There are no changes between Jakarta Bean Validation 2.0 and Bean Validation 2.0 except for the GAV: it is now
`jakarta.validation:jakarta.validation-api`.

### Certified implementations - `HibernateValidator:6.0.17.Final`

### Jakarta Bean Validation 文档

* [Jakarta Bean Validation specification 2.0](https://beanvalidation.org/2.0/spec/)

* 非官方参考
    * [@Validated和@Valid的区别？教你使用它完成Controller参数校验](https://blog.csdn.net/f641385712/article/details/97621783)
    * [让Controller支持对平铺参数执行数据校验（默认Spring MVC使用@Valid只能对JavaBean进行校验）](https://blog.csdn.net/f641385712/article/details/97621755)
    * [SpringBoot中BeanValidation数据校验与优雅处理详解](https://www.cnblogs.com/summerday152/p/13984576.html)

## Hibernate Validator

### Annotation Processor

Hibernate Validator Annotation Processor is the right thing for you. It helps preventing such mistakes by plugging
into the build process and raising compilation errors whenever constraint annotations are incorrectly used.

支持多种形式使用：`Maven`/`Gradle`/`javac`/`Apache Ant`/`Eclipse`/`IntelliJ IDEA`/`NetBeans`

> [Annotation Processor](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-annotation-processor)

### Hibernate Validator 文档

* [官网](https://hibernate.org/validator/)
* [Hibernate Validator 6.1 官方文档](https://docs.jboss.org/hibernate/validator/6.1/reference/en-US/html_single/)
* [占位符配置](https://docs.jboss.org/hibernate/validator/6.1/reference/en-US/html_single/#section-interpolation-with-message-expressions)
  * 属性: `{min}`
  * 当前验证值: `${validatedValue}`
  * 表达式: `${value > 1 ? 's' : ''}`
  * 自定义: `${formatter.format('%1$.2f', validatedValue)}`

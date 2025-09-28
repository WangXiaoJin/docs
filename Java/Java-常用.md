# Java 常用

## 1. Jdk

### 时区

通过 `java.time.format.ZoneName` 查阅时区

### Javadoc

JavaDoc命令详解及常用注释详解
* [Jdk7 JavaDoc 官方文档](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javadoc.html)
* [Jdk8 JavaDoc 官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javadoc.html)
* [Jdk17 JavaDoc 官方文档](https://docs.oracle.com/en/java/javase/17/docs/specs/javadoc/doc-comment-spec.html)

#### Doclet

Doclets are programs written in the Java™ programming language that use the doclet API to specify the content and
 format of the output of the Javadoc tool.
[Doclet Overview](https://docs.oracle.com/javase/7/docs/technotes/guides/javadoc/doclet/overview.html)

#### Taglet

Taglets are programs written in the Java™ programming language that implement the Taglet API. Taglets can be written as
either block tags, such as `@todo`, or inline tags, such as `{@underline}`.
[Taglet Overview](https://docs.oracle.com/javase/7/docs/technotes/guides/javadoc/taglet/overview.html)

* [Javadoc Technology](https://docs.oracle.com/javase/7/docs/technotes/guides/javadoc/index.html)
* [javadoc - The Java API Documentation Generator](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javadoc.html) - 详解
* [A Guide to Formatting Code Snippets in Javadoc](https://reflectoring.io/howto-format-code-snippets-in-javadoc/) - `<pre>`/`{@code}`/`<code>`


### Java 获取 `Content-Type` / `MimeType`

* `java.net.URLConnection.guessContentTypeFromName(fileName)`
* `java.net.URLConnection.guessContentTypeFromStream(InputStream)`
* `java.nio.file.Files.probeContentType(Path)`
* `org.springframework.http.MediaTypeFactory.getMediaType(org.springframework.core.io.Resource)`
* `org.springframework.http.MediaTypeFactory.getMediaType(java.lang.String)`

> [Getting a File’s Mime Type in Java](https://www.baeldung.com/java-file-mime-type)

### Java编译后获取方法的参数名

* [为何Spring MVC可获取到方法参数名，而MyBatis却不行？](https://cloud.tencent.com/developer/article/1497751)

### Unicode Normalize - `NFC`/`NFD`/`NFKC`/`NFKD`

* [Difference Between NFD, NFC, NFKD, and NFKC Explained with Python Code](https://towardsdatascience.com/difference-between-nfd-nfc-nfkd-and-nfkc-explained-with-python-code-e2631f96ae6c)
* [从⽅不是方到Unicode正规化NFD, NFC, NFKD, NFKC](https://xobo.org/unicode-normalization-nfd-nfc-nfkd-nfkc/)
* [`String.prototype.normalize()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/normalize)

### Character

* [Character#isAlphabetic() VS Character#isLetter()](https://www.baeldung.com/java-character-isletter-isalphabetic)
  * `UPPERCASE_LETTER`
  * `LOWERCASE_LETTER`
  * `TITLECASE_LETTER`
  * `MODIFIER_LETTER`
  * `OTHER_LETTER`
  * `LETTER_NUMBER`

### 正则 - RegExp

* [正则表达式 - 可视化图解](https://regexper.com/)

#### Google RE2

谷歌开源的正则表达式库，性能较好。

* [RE2 语法](https://github.com/google/re2/wiki/Syntax)


### JDK sun 包源码

* [IDEA查看Java的sun包下的源码](https://plentymore.github.io/2019/01/04/IDEA%E6%9F%A5%E7%9C%8BJava%E7%9A%84sun%E5%8C%85%E4%B8%8B%E7%9A%84%E6%BA%90%E7%A0%81/)
* [idea中查看sun.misc中的源码](https://www.cnblogs.com/gingo/p/14805160.html)

#### Unsafe 类

* [Java魔法类：Unsafe应用解析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html) - 美团技术团队
* [JAVA之Unsafe学习笔记](https://www.jianshu.com/p/b4de97ca8bb9)

### 设计模式

* [设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html) - 各设计模式示例


### 并发

* [Java中`notify`和`notifyAll`有什么区别？](https://www.zhihu.com/question/37601861)

* ForkJoinPool
  * [ForkJoinPool大型图文现场（一阅到底 vs 直接收藏）](https://segmentfault.com/a/1190000039267451)
  * [Java 并发编程笔记：如何使用 ForkJoinPool 以及原理](https://blog.dyngr.com/blog/2016/09/15/java-forkjoinpool-internals/)
  * [Java fork/join 框架用法及原理](https://baeldung-cn.com/java-fork-join) - Baeldung

* CompletableFuture
  * [Java CompletableFuture 详解](https://colobu.com/2016/02/29/Java-CompletableFuture/)
  * [Guide To CompletableFuture](https://www.baeldung.com/java-completablefuture)

### Synthetic / Bridge

* Synthetic Fields
* Synthetic Methods
* Synthetic Constructors
* Bridge Methods
> [Synthetic Constructs in Java - 参考链接](https://www.baeldung.com/java-synthetic)

### Java各版本新特性

* [JDK Enhancement Proposals - JEP索引排序（官网）](https://openjdk.org/jeps/0)
* [JDK Enhancement Proposals - Java版本排序（官网）](https://openjdk.org/projects/jdk/)
* [各版本新特性 - 简易参考](https://pdai.tech/md/java/java8up/java9.html)
* module（模块化）
  * [Module 定义](https://docs.oracle.com/javase/specs/jls/se17/html/jls-7.html#jls-7.7) - Java官方文档
  * [Automatic Module](https://stackoverflow.com/questions/46741907/what-is-an-automatic-module)
  * [Java 9 的模块（Module）系统](https://juejin.cn/post/7078562146115649544)
  * [关于 Java 模块系统，看这一篇就够了](https://www.51cto.com/article/620291.html)
  * [Module - 廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1281795926523938#0)
* Java 9（内容质量教好，值得推荐）
  * [2. 模块化系统](https://www.cnblogs.com/IcanFixIt/p/6947763.html)
  * [3. 创建你的第一个模块](https://www.cnblogs.com/IcanFixIt/p/6956514.html)
  * [4. 模块依赖](https://www.cnblogs.com/IcanFixIt/p/6994501.html)
  * [5. 实现服务](https://www.cnblogs.com/IcanFixIt/p/7050433.html)
  * [6. 封装模块](https://www.cnblogs.com/IcanFixIt/p/7095839.html)
    * JMOD格式
    * 多版本JAR
  * [7. 创建自定义运行时映像](https://www.cnblogs.com/IcanFixIt/p/7110790.html)
    * 自定义运行时映像和 jimage 格式
    * jlink
  * [8. JDK 9重大改变](https://www.cnblogs.com/IcanFixIt/p/7131676.html)
    * 新的JDK版本控制方案
    * JDK和JRE的改变
    * 支持标准覆盖机制
    * 扩展机制、类加载器
    * 访问资源
    * 使用JDK内部API
    * 修补模块内容
  * [9. 打破模块封装](https://www.cnblogs.com/IcanFixIt/p/7144366.html)
  * [10. 模块相关API](https://www.cnblogs.com/IcanFixIt/p/7161615.html)
    * 查询模块
    * 更新模块
    * 模块声明及属性
    * 模块层
  * [15. 增强的弃用注解](https://www.cnblogs.com/IcanFixIt/p/7234054.html)
    * @Deprecation
    * jdeprscan
  * [16. 虚拟机栈遍历](https://www.cnblogs.com/IcanFixIt/p/7238835.html)
  * [17. Reactive Streams](https://www.cnblogs.com/IcanFixIt/p/7245377.html)
  * [19. 平台和JVM日志](https://www.cnblogs.com/IcanFixIt/p/7259712.html)
  * [20. JDK 9中API层次的改变](https://www.cnblogs.com/IcanFixIt/p/7271461.html)
    * 下划线作为新关键字
    * 改进使用`try-with-resources`块的语法
    * 如何在匿名类中使用`<>`操作符
    * 如何在接口中使用私有方法
    * 如何在私有方法上使用`@SafeVarargs`注解
    * 如何丢弃子进程的输出
    * 如何在`Math`和`StrictMath`类中使用新的方法
    * 如何使用`Optionals`流以及`Optionals`上的新的操作
    * 如何使用等待提示（`spin-wait hints`）
    * 对Time API和Matcher和Objects类的增强
    * 如何比较数组和数组的一部分
    * Javadoc的增强功能以及如何使用其新的搜索功能
    * 本地桌面支持JDK 9以及如何使用它们
    * 在对象反序列化过程中如何使用全局和局部过滤器
    * 如何将数据从输入流传输到输出流以及如何复制和分片缓冲区


## 2. 位运算

* [原码、反码、补码的产生、应用以及优缺点有哪些？](https://www.zhihu.com/question/20159860) - 值得一读


## 3. 文档

* [官方文档](https://docs.oracle.com/en/java/javase/) - 可选择你需要的JDK版本

* [Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html) -【重要】

* [重写 `equals` 方法注意事项](https://www.artima.com/lejava/articles/equality.html)

* [主流浏览器附件下载中文乱码](http://www.zhushiyao.com/?p=5017)

* SecureRandom
    * [The Right Way to Use SecureRandom](https://tersesystems.com/blog/2015/12/17/the-right-way-to-use-securerandom/)
    * [使用 SecureRandom 产生随机数采坑记录](https://cloud.tencent.com/developer/article/1558293)
# Java 常用

## Jdk

### 时区

通过 `java.time.format.ZoneName` 查阅时区

### Javadoc

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

## 位运算

* [原码、反码、补码的产生、应用以及优缺点有哪些？](https://www.zhihu.com/question/20159860) - 值得一读


## 文档

* [官方文档](https://docs.oracle.com/en/java/javase/) - 可选择你需要的JDK版本

* [Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html) -【重要】

* Java各版本新特性
  * [JDK Enhancement Proposals - JEP索引排序（官网）](https://openjdk.org/jeps/0)
  * [JDK Enhancement Proposals - Java版本排序（官网）](https://openjdk.org/projects/jdk/)
  * [各版本新特性 - 简易参考](https://pdai.tech/md/java/java8up/java9.html)
  * module（模块化）
    * [Java 9 的模块（Module）系统](https://juejin.cn/post/7078562146115649544)
    * [关于 Java 模块系统，看这一篇就够了](https://www.51cto.com/article/620291.html)
    * [Module - 廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1281795926523938#0)

* [重写 `equals` 方法注意事项](https://www.artima.com/lejava/articles/equality.html)

* [主流浏览器附件下载中文乱码](http://www.zhushiyao.com/?p=5017)

* SecureRandom
    * [The Right Way to Use SecureRandom](https://tersesystems.com/blog/2015/12/17/the-right-way-to-use-securerandom/)
    * [使用 SecureRandom 产生随机数采坑记录](https://cloud.tencent.com/developer/article/1558293)
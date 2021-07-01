# Java 常用

## Jdk

### 时区

通过 `java.time.format.ZoneName` 查阅时区

### Javadoc Technology

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

### Java 获取 `Content-Type` / `MimeType`

* `java.net.URLConnection.guessContentTypeFromName(fileName)`
* `java.net.URLConnection.guessContentTypeFromStream(InputStream)`
* `java.nio.file.Files.probeContentType(Path)`
* `org.springframework.http.MediaTypeFactory.getMediaType(org.springframework.core.io.Resource)`
* `org.springframework.http.MediaTypeFactory.getMediaType(java.lang.String)`

> [Getting a File’s Mime Type in Java](https://www.baeldung.com/java-file-mime-type)

### Java编译后获取方法的参数名

* [为何Spring MVC可获取到方法参数名，而MyBatis却不行？](https://cloud.tencent.com/developer/article/1497751)

## 文档

* [官方文档](https://docs.oracle.com/en/java/javase/) - 可选择你需要的JDK版本

* [Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html) -【重要】

* [重写 `equals` 方法注意事项](https://www.artima.com/lejava/articles/equality.html)

* [主流浏览器附件下载中文乱码](http://www.zhushiyao.com/?p=5017)

* SecureRandom
    * [The Right Way to Use SecureRandom](https://tersesystems.com/blog/2015/12/17/the-right-way-to-use-securerandom/)
    * [使用 SecureRandom 产生随机数采坑记录](https://cloud.tencent.com/developer/article/1558293)
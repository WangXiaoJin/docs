# APT-AnnotationProcessingTool

注解处理器（Annotation Processor）是javac的一个工具，它用来在编译时扫描和处理注解（Annotation）。

一个注解的注解处理器，以Java代码（或者编译过的字节码）作为输入，生成文件（通常是.java文件）作为输出。
这具体的含义什么呢？你可以生成Java代码！这些生成的Java代码是在生成的.java文件中，所以你不能修改已经存在的Java类，
例如向已有的类中添加方法。这些生成的Java文件，会同其他普通的手动编写的Java源代码一样被javac编译。

An important thing to note is `the limitation of the annotation processing API — it can only be used to generate new
files, not to change existing ones`.

The notable exception is the `Lombok` library which uses annotation processing as a bootstrapping mechanism to include
itself into the compilation process and modify the AST via some internal compiler APIs. This hacky technique has nothing
to do with the intended purpose of annotation processing and therefore is not discussed in this article.


## 文档

* [Java注解处理器](https://race604.com/annotation-processing/)
* [Java Annotation Processing and Creating a Builder](https://www.baeldung.com/java-annotation-processing-builder)
* [Using annotation processor in IDE](https://immutables.github.io/apt.html)
















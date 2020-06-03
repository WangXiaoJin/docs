# Java 常用

## Jdk

* 通过 `java.time.format.ZoneName` 查阅时区

* HotSpot虚拟机提供了很对非稳定参数（Unstable Options，即以`-XX:`开头的参数，JDK1.6的虚拟机中大概有660多个），
使用`-XX:+PrintFlagsFinal`参数可以输出所有参数的名称及默认值（默认不包含Diagnostic和Experimental的参数），
如果需要，可以配合`-XX:+UnlockDiagnosticVMOptions/-XX:+UnlockExperimentalVMOptions`一起使用）。参数的使用方法：
    * `-XX:+<option>` 开启option参数
    * `-XX:-<option>` 关闭option参数
    * `-XX:<option>=<value>` 将option参数的值设置为value

## 文档

* [官方文档](https://docs.oracle.com/en/java/javase/) - 可选择你需要的JDK版本

* [Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html) -【重要】

* [深入理解Java虚拟机--Java对象的内存结构](https://blog.csdn.net/pengjunlee/article/details/72758619)

* [重写 `equals` 方法注意事项](https://www.artima.com/lejava/articles/equality.html)
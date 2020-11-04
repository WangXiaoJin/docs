# JVM

## 常用

* HotSpot虚拟机提供了很对非稳定参数（Unstable Options，即以`-XX:`开头的参数，JDK1.6的虚拟机中大概有660多个），
使用`-XX:+PrintFlagsFinal`参数可以输出所有参数的名称及默认值（默认不包含Diagnostic和Experimental的参数），
如果需要，可以配合`-XX:+UnlockDiagnosticVMOptions/-XX:+UnlockExperimentalVMOptions`一起使用）。参数的使用方法：
    * `-XX:+<option>` 开启option参数
    * `-XX:-<option>` 关闭option参数
    * `-XX:<option>=<value>` 将option参数的值设置为value


## 参考文档

### 综合

* [深入理解Java虚拟机--Java对象的内存结构](https://blog.csdn.net/pengjunlee/article/details/72758619)
* [The class File Format](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)

### Safepoint
* [Safepoints: Meaning, Side Effects and Overheads](http://psy-lob-saw.blogspot.com/2015/12/safepoints.html)
* [Where is my safepoint?](https://psy-lob-saw.blogspot.com/2014/03/where-is-my-safepoint.html)
* [Wait For It: Counted/Uncounted loops, Safepoints and OSR Compilation](http://psy-lob-saw.blogspot.com/2016/02/wait-for-it-counteduncounted-loops.html)
* [Why (Most) Sampling Java Profilers Are Fucking Terrible](http://psy-lob-saw.blogspot.com/2016/02/why-most-sampling-java-profilers-are.html)
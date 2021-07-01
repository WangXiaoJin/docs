# JVM

## 常用

* HotSpot虚拟机提供了很对非稳定参数（Unstable Options，即以`-XX:`开头的参数，JDK1.6的虚拟机中大概有660多个），
使用`-XX:+PrintFlagsFinal`参数可以输出所有参数的名称及默认值（默认不包含Diagnostic和Experimental的参数），
如果需要，可以配合`-XX:+UnlockDiagnosticVMOptions/-XX:+UnlockExperimentalVMOptions`一起使用）。参数的使用方法：
    * `-XX:+<option>` 开启option参数
    * `-XX:-<option>` 关闭option参数
    * `-XX:<option>=<value>` 将option参数的值设置为value

* `java -XX:+PrintCommandLineFlags -version`/`java -XX:+PrintFlagsFinal -version` - 输出参数

* [Eclipse OpenJ9 Docs - 命令行参数详解](https://www.eclipse.org/openj9/docs/xxusecontainersupport/)

* [JVM Options - The complete reference](http://pingtimeout.github.io/jvm-options/#)

* `容器应用`的`CPU`/`内存`
    * [Docker support in Java 8 — finally!](https://blog.softwaremill.com/docker-support-in-new-java-8-finally-fd595df0ca54)
    * [+UseContainerSupport to the Rescue - 新](https://medium.com/adorsys/usecontainersupport-to-the-rescue-e77d6cfea712)
    * [JVM Memory Settings in a Container Environment - 旧](https://medium.com/adorsys/jvm-memory-settings-in-a-container-environment-64b0840e1d9e)

## Heap & GC

### GC配置

常用配置：
```
-XX:+AlwaysPreTouch

-verbose:gc
-Xloggc:gc.log
-XX:+PrintGC

-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintGCDetails
-XX:+PrintTenuringDistribution
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintHeapAtGC

-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=50
-XX:GCLogFileSize=64m

-XX:+PrintClassHistogramAfterFullGC
-XX:+PrintClassHistogramBeforeFullGC

# 检测因Safepoint导致应用停顿时间较长，输出到控制台，不会输出到gc日志文件中
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1
-XX:+SafepointTimeout
-XX:SafepointTimeoutDelay=<ms before timeout log is printed>
```

* 应用重启后GC日志丢失问题
    * [GC log rotation data lose on application restart](https://stackoverflow.com/questions/19274153/gc-log-rotation-data-lose-on-application-restart)
    * [Make the GC log file name parameterized](https://bugs.openjdk.java.net/browse/JDK-6950794)

* GC
    * G1
        * [Garbage First Garbage Collector Tuning](https://www.oracle.com/technical-resources/articles/java/g1gc.html)
    * CMS
        * [Understanding CMS GC Logs](https://blogs.oracle.com/poonam/understanding-cms-gc-logs)
        * [The Unspoken - CMS and PrintGCDetails](https://blogs.oracle.com/jonthecollector/the-unspoken-cms-and-printgcdetails)

* Java Garbage Collection - 系列推荐
    * [A Quick Start on Java Garbage Collection: What It Is & How It Works](https://sematext.com/blog/java-garbage-collection/)
    * [Understanding Java Garbage Collection Logging: What Are GC Logs and How To Analyze Them](https://sematext.com/blog/java-garbage-collection-logs/)
    * [A Step-by-Step Guide to Java Garbage Collection Tuning](https://sematext.com/blog/java-garbage-collection-tuning/)
* [Garbage Collection Tuning Guide, Release 8 - 官网](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/index.html)
    * [Java13 - 官网](https://docs.oracle.com/en/java/javase/13/gctuning/index.html)
* [Garbage Collection Logging to a File in Java - GC日志配置](https://www.baeldung.com/java-gc-logging-to-file)
* [Understanding the Java Garbage Collection Log - 理解GC日志](https://dzone.com/articles/understanding-garbage-collection-log)
* [GC Logging – user, sys, real – which time to use?](https://blog.tier1app.com/2016/04/06/gc-logging-user-sys-real-which-time-to-use/)
* [JVM Pauses - It's more than GC - `Safepoint`导致应用暂停响应](https://blanco.io/blog/jvm-safepoint-pauses/)


### GC Tools

1. `GCViewer` - 开源工具
    * [GCViewer官网](https://www.tagtraum.com/gcviewer.html)
    * [GCViewer - Github](https://github.com/chewiebug/GCViewer)
    * [VMFlags](https://www.tagtraum.com/gcviewer-vmflags.html)
2. [gceasy - 在线GC分析](https://gceasy.io/)

### Heap Tools

1. [MAT](https://www.eclipse.org/mat/)
    * [MemoryAnalyzer/FAQ](http://wiki.eclipse.org/MemoryAnalyzer/FAQ)
    * [linux使用MAT分析dump文件](http://www.moheqionglin.com/site/blogs/84/detail.html)
    * [MemoryAnalyzer - WIKI](https://wiki.eclipse.org/MemoryAnalyzer)
        * `What if the Heap Dump is NOT Written on OutOfMemoryError?`
2. [IBM HeapAnalyzer](https://www.ibm.com/support/pages/ibm-heapanalyzer)
3. [YourKit Java Profiler](https://www.yourkit.com/)
4. [JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)
5. [heaphero.io](https://heaphero.io/)
6. [BHeapSampler](http://dr-brenschede.de/bheapsampler/)
7. [Java故障诊断和性能分析工具 -【综合文章】](http://www.jiaqili.me/post/java-profiling-tools/)

### 性能调优

1. [JVM Tuning: How to Prepare Your Environment for Performance Tuning](https://sematext.com/blog/jvm-performance-tuning/)
2. [Native Memory Tracking - 跟踪VM所有内存使用情况](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html) - 会带来5%-10%的性能损耗

## 综合

* [深入理解Java虚拟机--Java对象的内存结构](https://blog.csdn.net/pengjunlee/article/details/72758619)
* [The class File Format](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)

## Safepoint
* [Safepoints: Meaning, Side Effects and Overheads](http://psy-lob-saw.blogspot.com/2015/12/safepoints.html)
* [Where is my safepoint?](https://psy-lob-saw.blogspot.com/2014/03/where-is-my-safepoint.html)
* [Wait For It: Counted/Uncounted loops, Safepoints and OSR Compilation](http://psy-lob-saw.blogspot.com/2016/02/wait-for-it-counteduncounted-loops.html)
* [Why (Most) Sampling Java Profilers Are Fucking Terrible](http://psy-lob-saw.blogspot.com/2016/02/why-most-sampling-java-profilers-are.html)

## Metaspace
* [深入理解堆外内存 Metaspace](https://www.javadoop.com/post/metaspace)
* [What is Metaspace?](https://stuefe.de/posts/metaspace/what-is-metaspace/)



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

> JDK8 默认 `GC`/`Heap` - [官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html)

### GC

#### 垃圾收集器

**新生代**
1. `Copy（别名：Serial、DefNew）`- the serial copy collector, uses one thread to copy surviving objects from Eden to Survivor spaces and between Survivor spaces until it decides they've been there long enough, at which point it copies them into the old generation.
2. `PS Scavenge（别名：Parallel Scavenge、PSYoungGen）`- the parallel scavenge collector, like the Copy collector, but uses multiple threads in parallel and has some knowledge of how the old generation is collected (essentially written to work with the serial and PS old gen collectors).
3. `ParNew （别名和内部名一样为ParNew）`- the parallel copy collector, like the Copy collector, but uses multiple threads in parallel and has an internal 'callback' that allows an old generation collector to operate on the objects it collects (really written to work with the concurrent collector).
4. `G1 Young Generation` - the garbage first collector, uses the 'Garbage First' algorithm which splits up the heap into lots of smaller spaces, but these are still separated into Eden and Survivor spaces in the young generation for G1.

**老年代**
1. `MarkSweepCompact （别名：Serial Old（MSC））`- the serial mark-sweep collector, the daddy of them all, uses a serial (one thread) full mark-sweep garbage collection algorithm, with optional compaction.
2. `PS MarkSweep（别名：Parallel Old ）`- the parallel scavenge mark-sweep collector, parallelised version (i.e. uses multiple threads) of the MarkSweepCompact.
3. `ConcurrentMarkSweep （别名：CMS）`- the concurrent collector, a garbage collection algorithm that attempts to do most of the garbage collection work in the background without stopping application threads while it works (there are still phases where it has to stop application threads, but these phases are attempted to be kept to a minimum). Note if the concurrent collector fails to keep up with the garbage, it fails over to the serial MarkSweepCompact collector for (just) the next GC.
4. `G1 Mixed Generation` -  the garbage first collector, uses the 'Garbage First' algorithm which splits up the heap into lots of smaller spaces.

All of the garbage collection algorithms except `ConcurrentMarkSweep` are `stop-the-world`, i.e. they stop all application 
threads while they operate - the stop is known as 'pause' time. The ConcurrentMarkSweep tries to do most of it's work in 
the background and minimize the pause time, but it also has a stop-the-world phase and can fail into the MarkSweepCompact 
which is fully stop-the-world. (The G1 collector has a concurrent phase but is currently mostly stop-the-world).

> 参考链接 - [JVM垃圾收集器组合--各种组合对应的虚拟机参数实践](https://www.cnblogs.com/grey-wolf/p/10222758.html)

#### 新生代老年代之间的收集器组合

That's the set of garbage collectors available, but they operate in two different heap spaces and it's the combination 
that is what we actually end up with for a particular JVM setting, so I'll also list the combinations that are possible. 
It doesn't explode into a dozen combinations because not all of these collectors work with each other. 
G1 is effectively an antisocial collector that doesn't like working with anyone else; the serial collectors are the 
"last man picked" collectors; the 'PS' collectors like to work with each other; and ParNew and Concurrent like to work together.

The full list of possible GC algorithm combinations that can work are:

|  Command Options  | Resulting Collector Combination  | 
|:--------: |:----------: |
| `-XX:+UseSerialGC` | young `Copy` and old `MarkSweepCompact` |
| `-XX:+UseG1GC` | young `G1 Young` and old `G1 Mixed` |
| `-XX:+UseParallelGC -XX:+UseParallelOldGC -XX:+UseAdaptiveSizePolicy` | young `PS Scavenge` old `PS MarkSweep` with adaptive sizing |
| `-XX:+UseParallelGC -XX:+UseParallelOldGC -XX:-UseAdaptiveSizePolicy` | young `PS Scavenge` old `PS MarkSweep`, no adaptive sizing |
| `-XX:+UseParNewGC` (deprecated in Java 8 and removed in Java 9 - for ParNew see the line below which is NOT deprecated) | young `ParNew` old `MarkSweepCompact` |
| `-XX:+UseConcMarkSweepGC -XX:+UseParNewGC` | young `ParNew` old `ConcurrentMarkSweep` |
| `-XX:+UseConcMarkSweepGC -XX:-UseParNewGC` (deprecated in Java 8 and removed in Java 9) | young `Copy` old `ConcurrentMarkSweep` |

All the combinations listed here will fail to let the JVM start if you add another GC algorithm not listed, 
with the exception of `-XX:+UseParNewGC` which is only combinable with `-XX:+UseConcMarkSweepGC`.

there are many many options for use with `-XX:+UseConcMarkSweepGC` which change the algorithm, e.g.
* `-XX:+/-CMSIncrementalMode` (deprecated in Java 8 and removed in Java 9) - uses or disables an incremental concurrent GC algorithm
* `-XX:+/-CMSConcurrentMTEnabled` - uses or disables parallel (multiple threads) concurrent GC algorithm
* `-XX:+/-UseCMSCompactAtFullCollection` - uses or disables a compaction when a full GC occurs

**其他选项:**

|  Command Options Used On Their Own  |                                                                    Equivalent To Entry In Table Above                                                                    | 
|:--------: |:------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| `-XX:+UseParallelGC` |                                                               `	-XX:+UseParallelGC -XX:+UseParallelOldGC`                                                                |
| `-XX:+UseParallelOldGC` |                                                                `-XX:+UseParallelGC -XX:+UseParallelOldGC`                                                                |
| `-Xincgc` (deprecated in Java 8 and removed in Java 9) |                                                                `-XX:+UseParNewGC -XX:+UseConcMarkSweepGC`                                                                |
| `-XX:+UseConcMarkSweepGC` |                                                                `-XX:+UseParNewGC -XX:+UseConcMarkSweepGC`                                                                |
| `-XX:+AggressiveHeap` | `-XX:+UseParallelGC -XX:+UseParallelOldGC -XX:+UseAdaptiveSizePolicy` with a bunch of other options related to sizing memory and threads and how they interact with the OS |

#### 通过程序输出使用的垃圾收集器

```java
import java.lang.management.GarbageCollectorMXBean;
import java.lang.management.ManagementFactory;
import java.util.List;

public class Test {
    public static void main(String[] args) {
        List<GarbageCollectorMXBean> l = ManagementFactory.getGarbageCollectorMXBeans();
        for (int i = 0; i < l.size(); i++) {
            GarbageCollectorMXBean garbageCollectorMXBean = l.get(i);
            if (i == 0) {
                System.out.println("young generation:" + garbageCollectorMXBean.getName());
            } else if (i == 1) {
                System.out.println("old generation:" + garbageCollectorMXBean.getName());
            }
        }
    }
}
```

#### 常用配置
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

#### 参考链接

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

### 查看 Metaspace 使用情况

#### 1. jmap -clstats pid

查看类加载器实例及已加载类的数量。

#### 2. JVM参数

* `-XX:+TraceClassLoading` - Enables tracing of classes as they are loaded. By default, this option is disabled and classes are not traced.
* `-XX:+TraceClassLoadingPreorder` - Enables tracing of all loaded classes in the order in which they are referenced. By default, this option is disabled and classes are not traced.
* `-XX:+TraceClassResolution` - Enables tracing of constant pool resolutions. By default, this option is disabled and constant pool resolutions are not traced.
* `-XX:+TraceClassUnloading` - Enables tracing of classes as they are unloaded. By default, this option is disabled and classes are not traced.
* `-XX:+TraceLoaderConstraints` - Enables tracing of the loader constraints recording. By default, this option is disabled and loader constraints recording is not traced.

#### 3. jcmd pid GC.class_stats

查看已加载的类信息：

`jcmd pid GC.class_stats |awk '{print $13}'| sort | uniq -c | sort -r`

> 注：`jcmd <pid> help` - 列出支持的参数列表。  
> GC.class_stats 需要启用`-XX:+UnlockDiagnosticVMOptions`功能

#### 4. Native Memory Tracking
通过 NMT 查看 堆及非堆的内存使用情况

* [Native Memory Tracking](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/nmt-8.html) - 启动 NMT
* [Native Memory Tracking](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html) - 使用手册
* [Native Memory Tracking 详解](https://heapdump.cn/article/4644018)

##### 错误：Native memory tracking is not enabled 

确保java进程启动时启用了`-XX:NativeMemoryTracking`。

##### 错误：Java HotSpot(TM) 64-Bit Server VM warning: Native Memory Tracking did not setup properly, using wrong launcher?

`-XX:NativeMemoryTracking` 需放在`-jar`指令前面。[参考文档](https://stackoverflow.com/questions/53955159/not-able-to-enable-native-memory-tracking-in-a-spring-boot-application)



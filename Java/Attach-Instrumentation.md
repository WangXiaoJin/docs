# Attach Instrumentation

使用 Instrumentation，使得开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，
甚至能够替换和修改某些类的定义。

## [Byte Buddy Agent](https://bytebuddy.net/#/tutorial)

The Byte Buddy agent provides a JVM Instrumentation in order to allow Byte Buddy the redefinition of already loaded classes.

使用`Instrumentation instrumentation = ByteBuddyAgent.install()`，可以在当前程序中获取`Instrumentation`来重定义Class，
或者直接使用`ByteBuddy`重定义Class：

```java
ByteBuddyAgent.install();

Foo foo = new Foo();
new ByteBuddy()
  .redefine(Bar.class)
  .name(Foo.class.getName())
  .make()
  .load(Foo.class.getClassLoader(), ClassReloadingStrategy.fromInstalledAgent());
assertThat(foo.m(), is("bar"));
```


## 文档

* JavaDoc
    * [VirtualMachine - JavaDoc](https://docs.oracle.com/javase/7/docs/jdk/api/attach/spec/com/sun/tools/attach/VirtualMachine.html)
    * [Package java.lang.instrument - JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)

* [Java Attach机制](https://www.jianshu.com/p/542e50edc8e3)

* [javaagent使用指南](https://www.cnblogs.com/rickiyang/p/11368932.html)
* [基于Java Instrument的Agent实现](https://www.jianshu.com/p/b72f66da679f)
* [Java程序员必知：深入理解Instrument](https://www.jianshu.com/p/5c62b71fd882)
* [JVM源码分析之javaagent原理完全解读](https://www.infoq.cn/article/javaagent-illustrated)
* Java Agent入门实战
    * [Java Agent入门实战（一）-Instrumentation介绍与使用](https://juejin.im/post/6844904035305127950)
    * [Java Agent入门实战（二）-Instrumentation源码概述](https://juejin.im/post/6844904038622838797)
    * [Java Agent入门实战（三）-JVM Attach原理与使用](https://juejin.im/post/6844904039830781966)
















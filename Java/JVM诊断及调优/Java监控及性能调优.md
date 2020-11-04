# Java监控及性能调优

## 常用工具

### [async-profiler](https://github.com/jvm-profiling-tools/async-profiler)

This project is a low overhead sampling profiler for Java that does not suffer from Safepoint bias problem.

async-profiler can trace the following kinds of events:
* CPU cycles
* Hardware and Software performance counters like cache misses, branch misses, page faults, context switches etc.
* Allocations in Java Heap
* Contented lock attempts, including both Java object monitors and ReentrantLocks


### Bistoury

Bistoury 是去哪儿网开源的一个对应用透明，无侵入的java应用诊断工具，用于提升开发人员的诊断效率和能力。

Bistoury 的目标是一站式java应用诊断解决方案，让开发人员无需登录机器或修改系统，就可以从日志、内存、线程、类信息、调试、机器和
系统属性等各个方面对应用进行诊断，提升开发人员诊断问题的效率和能力。集成Alibaba开源的arthas和唯品会开源的vjtools，提供了更加丰富的功能。

* [Bistoury - Github](https://github.com/qunarcorp/bistoury)
    * [设计文档](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/design/design.md)
    * [快速开始](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/quick_start.md) - 本地测试使用，不可用于生产环境
    * [生产部署](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/deploy.md)
    * [git及maven配置](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/gitlab_maven.md)
    * [在线debug](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/debug.md)
    * [线程级cpu使用率监控](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/jstack.md)
    * [命令使用文档](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/commands.md)
    * [动态监控](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/monitor.md)
    * [应用中心](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/application.md)
    * [应用pid获取](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/PID.md)
    * [文件下载](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/downloadFile.md)
    * [性能分析](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/profiler.md)
    * [jmap](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/jmap.md)
    * [Bistoury支持Java 11](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/java11.md)
    * [bistoury存储方案](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/store.md)
    * [常见问题汇总](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/FAQ.md)
        * 日志目录
        * not find proxy for agent
        * 获取ip错误
        * agent attach时加载初始化类失败
        * windows 环境暂时不支持
        * 在线debug暂时不支持github仓库的源码调试
        * jdk版本
        * 端口问题
        * 在线debug时,前端界面上对象值显示 `object size greater than ***kb`
        * 在线debug时 前端界面右上角提示 *** 代码查看仅可通过反编译
        * 在线debug添加断点出现 `register breakpoint fail, File was not found in the executable`
        * 动态监控添加监控出现 `add monitor failed, File was not found in the executable`
* [java应用诊断和在线debug利器bistoury介绍与在K8S环境使用](https://www.cnblogs.com/xiaoqi/p/Bistoury.html)

日志目录：

|      模块            | 输出的具体文件夹               |
|:---------------------|:------------------------------|
| 1. agent          | 解压缩目录/bistoury-agent-bin/logs |
| 2. proxy         | 解压缩目录/bistoury-proxy-bin/logs |
| 3. ui             | 解压缩目录/bistoury-ui-bin/logs    |
| 4. attach到应用中的部分 | 应用进程所属用户的主目录/logs/   |

部署步骤：
1. 下载`bistoury-2.0.7`源码：`wget https://github.com/qunarcorp/bistoury/archive/v2.0.7.zip`
2. 构建：`cd script && ./build.sh`
    * `bistoury-agent` - `bistoury-dist/target/bistoury-agent-bin.tar.gz`
    * `bistoury-proxy` - `bistoury-proxy/target/bistoury-proxy-bin.tar.gz`
    * `bistoury-ui` - `bistoury-ui/target/bistoury-ui-bin.tar.gz`
    > 构建的目的是为了启用`prod` `profile`，从而使用`mysql`而非`h2`
3. 初始化SQL：`bistoury-ui/sql/bistoury_init.sql` ，初始化后用户名：admin、密码：admin
4. 修改JDBC连接配置：`bistoury-ui/conf/jdbc.properties`、`bistoury-proxy/conf/jdbc.properties`
5. 修改`bistoury-proxy`项目icon： `bistoury-proxy/webapp/favicon.ico` / `bistoury-ui/webapp/favicon.ico`
6. 修改获取`pid`规则: [pid获取](https://github.com/qunarcorp/bistoury/blob/d5cf37ac35282cb19074006c844354e48f5ad053/docs/cn/PID.md)
7. 修改文件下载目录: `bistoury-proxy/conf/download_dir_limit.properties` - 可配置全局及针对单个应用，应用中心也可自定义配置
8. 配置ZK地址：`bistoury-proxy/conf/registry.properties`、`bistoury-ui/conf/registry.properties`
9. 修改JAVA_HOME配置
    * `bistoury-proxy/bin/bistoury-proxy-env.sh`
    * `bistoury-ui/bin/bistoury-ui-env.sh`
    * `bistoury-agent/bin/bistoury-agent-env.sh`
10. [配置`bistoury-agent`](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/deploy.md#233-bistoury-agent%E9%83%A8%E7%BD%B2) - 【重要】
    * `bistoury-agent-env.sh` - `BISTOURY_PROXY_HOST="192.168.200.121:9090"` 域名前不能带`http://`，因为它会自己增加前缀
    * 存储方案从rocksdb换为sqlite - `-Dbistoury.store.db=sqlite`
    * `bistoury.app.lib.class`
    * 其他配置参考文档
11. 启动服务
    * `bistoury-proxy`
        * 启动 - `./bistoury-proxy.sh start`
        * 停止 - `./bistoury-proxy.sh stop`
        * 重写 - `./bistoury-proxy.sh restart`
    * `bistoury-ui`
        * 启动 - `./bistoury-ui.sh start`
        * 停止 - `./bistoury-ui.sh stop`
        * 重写 - `./bistoury-ui.sh restart`
    * `bistoury-agent`
        * 启动 - `./bistoury-agent.sh -p 100 start` / `./bistoury-agent.sh start`
        * 停止 - `./bistoury-agent.sh stop`
        * 重写 - `./bistoury-agent.sh -p 101 restart` / `./bistoury-agent.sh restart`

    > 解压并调整完配置后运行bin目录下的脚本进行启动，可以在`bistoury-proxy-env.sh`/`bistoury-agent-env.sh`/`bistoury-ui-env.sh`
    中的JAVA_OPTS里配置JVM相关参数，GC相关配置已配置。

    > `bistoury-ui`默认端口为`9091`, 因此启动成功以后可以访问`http://127.0.0.1:9091`访问ui页面，用户名密码默认都为`admin`。

12. 低版本SpringBoot（如：1.3）打出的jar包中`MANIFEST.MF`不包含`Spring-Boot-Classes`/`Spring-Boot-Lib`，且目录结构不同，
低版本的SpringBoot应用启动时需要增加属性：`-Dbistoury.jar.lib.path=/tmp/bistoury/store/bistoury_tomcat_webapp/lib/ -Dbistoury.jar.source.path=/tmp/bistoury/store/bistoury_tomcat_webapp/com/`
    * `bistoury.jar.lib.path` - 依赖包目录(默认值`BOOT-INF/lib/`)
    * `bistoury.jar.source.path` - classes目录（默认值`BOOT-INF/classes/`）
    * `/tmp/bistoury/store/bistoury_tomcat_webapp` - 包默认解压目录

    > 这两个属性需要业务应用配置的原因是bistoury-agent应用在attach时并没有把这两个参数传递给业务应用，只传递了`bistoury.app.lib.class`
    ，只有`bistoury.app.lib.class`属性可以在bistoury-agent启动脚本里面配置：

    ```java
    // 在 qunar.tc.bistoury.commands.arthas.ArthasStarter#attachAgent() 方法里
    virtualMachine.loadAgent(realAgentFile.getCanonicalPath(),
        configure.getArthasCore() + delimiter + ";;" + configure.toString() + delimiter + System.getProperty("bistoury.app.lib.class"));
    ```

> 参考文档 - [生产部署](https://github.com/qunarcorp/bistoury/blob/master/docs/cn/deploy.md)


### [Arthas](https://arthas.aliyun.com/doc/)

* [Arthas里 Trace 命令怎样工作的/ Trace命令的实现原理](https://github.com/alibaba/arthas/issues/597)
* [Arthas实践--快速排查Spring Boot应用404/401问题](https://github.com/alibaba/arthas/issues/429)
* [Arthas实践--jad/mc/redefine线上热更新一条龙](http://hengyunabc.github.io/arthas-online-hotswap/)

* 表达式
    * [OGNL表达式官方指南](https://commons.apache.org/proper/commons-ognl/language-guide.html)
    * [OGNL特殊用法请参考](https://github.com/alibaba/arthas/issues/71)
    * [Arthas获取Spring Context](https://github.com/alibaba/arthas/issues/482)
    * [表达式核心变量](https://arthas.aliyun.com/doc/advice-class.html)
    * [Arthas问题排查集 - 活用ognl表达式](https://github.com/alibaba/arthas/issues/11)

* [Arthas IDEA Plugin](https://arthas.aliyun.com/doc/idea-plugin.html)
    * [Arthas IDEA Plugin 使用文档](https://www.yuque.com/docs/share/fa77c7b4-c016-4de6-9fa3-58ef25a97948?#)
    * [arthas idea plugin 文档 -【全】](https://www.yuque.com/wangji-yunque/ikhsmq)

* [profiler](https://arthas.aliyun.com/doc/profiler.html)
    * [async-profiler](https://github.com/jvm-profiling-tools/async-profiler)

* [用户案例](https://github.com/alibaba/arthas/issues?q=label%3Auser-case) - 包含常用的使用场景

* [Arthas依赖技术](https://github.com/alibaba/arthas/blob/master/README_CN.md#projects)


### [Arthas-MVEL](https://github.com/XhinLiang/arthas-mvel)

Arthas-MVEL use MVEL as first-class command parser and support all of the features of [Arthas](https://github.com/alibaba/arthas).

[MVEL](http://mvel.documentnode.com/) is a expression language just like OGNL but supports more features such as loop and function.


### [show-busy-java-threads](https://github.com/oldratlee/useful-scripts/blob/master/docs/java.md#-show-busy-java-threads)

纯脚本实现，简化命令：
* [show-busy-java-threads](https://github.com/oldratlee/useful-scripts/blob/master/docs/java.md#-show-busy-java-threads) -
用于快速排查Java的CPU性能问题(top us值过高)，自动查出运行的Java进程中消耗CPU多的线程。
* [show-duplicate-java-classes](https://github.com/oldratlee/useful-scripts/blob/master/docs/java.md#-show-duplicate-java-classes) -
找出Java Lib（Java库，即Jar文件）或Class目录（类目录）中的重复类。
* [find-in-jars](https://github.com/oldratlee/useful-scripts/blob/master/docs/java.md#-find-in-jars) -
在当前目录下所有jar文件里，查找类或资源文件。


## 参考文档

* [Tools and Commands Reference](https://docs.oracle.com/en/java/javase/12/tools/tools-and-command-reference.html)
  -【重要】Java官方所有命令详解

* [聊聊jvm的Code Cache](https://www.jianshu.com/p/b064274536ed)

* JOL - 分析`Java对象内存布局`，包括`基本使用`、`字节对齐`、`实例域重排序`、`继承`、`继承栅栏`、`继承对齐`等
  * [JVM基础 -- JOL使用教程 1](http://zhongmingmao.me/2016/07/01/jvm-jol-tutorial-1/)
  * [JVM基础 -- JOL使用教程 2](http://zhongmingmao.me/2016/07/02/jvm-jol-tutorial-2/)
  * [JVM基础 -- JOL使用教程 3](http://zhongmingmao.me/2016/07/03/jvm-jol-tutorial-3/)

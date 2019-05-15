## Maven Wrapper(统一Maven版本)

使用`Maven Wrapper`便于项目统一使用Maven版本号，当使用`Maven Wrapper`执行Maven命令时，首先在本地查找有无对应的
版本号的Maven，没有则下载，意味着你本地不需要提前安装Maven。

#### 安装

```bash
shell> cd yourmavenproject
shell> mvn -N io.takari:maven:wrapper
```

指定使用Maven版本号：
```bash
shell> cd yourmavenproject
shell> mvn -N io.takari:maven:wrapper -Dmaven=3.3.9
```

指定Maven的下载地址：
```bash
shell> cd yourmavenproject
shell> mvn -N io.takari:maven:wrapper -DdistributionUrl=http://server/path/to/maven/distro.zip
```

> 注：默认下载地址为：`https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.5.4/apache-maven-3.5.4-bin.zip`，
版本号不是固定的，此下载地址会比较慢，你可以替换成官网提供的镜像地址`http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.zip`。
替换URL地址有两种方案：1）通过`-DdistributionUrl=`参数 2）手动修改`maven-wrapper.properties`

> 注：`Maven Wrapper`支持缺少`.mvn/wrapper/maven-wrapper.jar`jar包，`mvnw`脚本发现缺少此jar，会通过`./mvn/wrapper/MavenWrapperDownloader.java`
下载jar包。下载的地址依据`maven-wrapper.properties`的`wrapperUrl`参数，没有此参数则使用`mvnw`脚本中的默认值。

#### 升级

* 再次执行安装命令
* 手动修改`.mvn/wrapper/maven-wrapper.properties`文件的`distributionUrl`参数

#### 使用

```bash
shell> ./mvnw clean install
```

> 注：用法和`mvn`完全一样

#### 文档

* [maven-wrapper Github地址](https://github.com/takari/maven-wrapper)
* [A Quick Guide to Maven Wrapper](https://www.baeldung.com/maven-wrapper)

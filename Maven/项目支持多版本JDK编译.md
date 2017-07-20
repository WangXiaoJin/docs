## 项目支持多版本JDK编译

* #### `maven-toolchains-plugin`插件实现（推荐）

    此插件将影响所有依赖JDK的插件
    
    `pom.xml`配置：
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <properties>
            <java.version>1.7</java.version>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <maven.compiler.source>${java.version}</maven.compiler.source>
            <maven.compiler.target>${java.version}</maven.compiler.target>
            <!--默认禁用toolchains插件-->
            <toolchains.phase>none</toolchains.phase>
        </properties>
        
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.1</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-toolchains-plugin</artifactId>
                    <version>1.1</version>
                    <executions>
                        <execution>
                            <phase>${toolchains.phase}</phase>
                            <goals>
                                <goal>toolchain</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <toolchains>
                            <jdk>
                                <version>${java.version}</version>
                                <vendor>sun</vendor>
                            </jdk>
                        </toolchains>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </project>
    ```
    
    `toolchains.xml`文件配置，参考Maven安装包提供的`toolchains.xml`文件：
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <toolchains xmlns="http://maven.apache.org/TOOLCHAINS/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/TOOLCHAINS/1.1.0 http://maven.apache.org/xsd/toolchains-1.1.0.xsd">
      <toolchain>
        <type>jdk</type>
        <provides>
          <version>1.7</version>
          <vendor>sun</vendor>
        </provides>
        <configuration>
          <jdkHome>C:\Program Files\Java\jdk1.7.0_101</jdkHome>
        </configuration>
      </toolchain>
      
      <toolchain>
        <type>jdk</type>
        <provides>
          <version>1.8</version>
          <vendor>sun</vendor>
        </provides>
        <configuration>
          <jdkHome>C:\Program Files\Java\jdk1.8.0_101</jdkHome>
        </configuration>
      </toolchain>
    </toolchains>
    ```
    
    `toolchains.xml`文件路径有两类型：
    * 用户级别：`${user.home}/.m2/toolchains.xml`，只针对当前用户。可通过命令行传参：`-t /path/to/user/toolchains.xml`
    * 全局级别：`${maven.home}/conf/toolchains.xml`，针对所有用户。可通过命令行传参：`-gt /path/to/global/toolchains.xml`
    
    > 注：通过修改属性配置`<toolchains.phase>validate</toolchains.phase>`来开启`toolchains`插件功能。  
        通过`<java.version>1.8</java.version>`来切换JDK版本。

* #### `maven-compiler-plugin`插件实现

    在执行`maven-compiler-plugin`插件时，配置JDK路径：
    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.6.1</version>
        <configuration>
            <!--<verbose>true</verbose>-->
            <!-- fork 配置需要设置为true -->
            <fork>true</fork>
            <!-- JDK路径 -->
            <executable>${JAVA_7_HOME}/bin/javac</executable>
            <!-- 编译器版本 -->
            <compilerVersion>1.3</compilerVersion>
        </configuration>
    </plugin>
    ```
    
    `JAVA_7_HOME`为自定义属性配置，常用案例如下：
    ```xml
    <settings>
      [...]
      <profiles>
        [...]
        <profile>
          <id>compiler</id>
            <properties>
              <JAVA_7_HOME>C:\Program Files\Java\jdk7</JAVA_1_4_HOME>
            </properties>
        </profile>
      </profiles>
      [...]
      <activeProfiles>
        <activeProfile>compiler</activeProfile>
      </activeProfiles>
    </settings>
    ```

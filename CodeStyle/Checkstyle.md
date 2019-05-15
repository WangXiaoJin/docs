## Checkstyle

#### Java Style and Configuration 

* [Google Java Style（规范）](http://checkstyle.sourceforge.net/reports/google-java-style-20170228.html)
* [Sun Code Conventions（规范）](http://www.oracle.com/technetwork/java/javase/documentation/codeconvtoc-136057.html)
* [Google Style Config（配置）](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml)
* [Sun Style Config（配置）](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/sun_checks.xml)
* [Checkstyle Style Config（配置）](https://github.com/checkstyle/checkstyle/blob/master/config/checkstyle_checks.xml)

#### Checkstyle

* 导入Google Code Style至IDE
    * IDEA - 从仓库<https://github.com/google/styleguide>中下载`intellij-java-google-style.xml`
    
        `File -> Settings -> Editor -> Code Style -> 点击齿轮图标 -> Import Scheme -> Intellij IDEA code style XML -> 选中刚才下载的配置文件`
    
    * Eclipse - 从仓库<https://github.com/google/styleguide>中下载`eclipse-java-google-style.xml`
    
        `Window -> Preferences -> Java -> Code Style -> Formatter -> 点击Import -> 选中刚下载的配置文件`

* IDE安装Checkstyle插件
    * IDEA - `File -> Settings -> Plugins -> Browse Repositories -> 搜索CheckStyle-IDEA -> 点击Install`
        * 配置`Checkstyle version/Scan scope/Configuration File` : `File | Settings | Other Settings | Checkstyle`
        * 配置时时检查代码 : `File | Settings | Editor | Inspections -> Checkstyle | Checkstyle real-time scan`
        * 执行Checkstyle
            * 右键当前文件 -> Check Current File
            * CheckStyle视窗操作
            * 使用IDEA自带的Inspect Code功能
    * Eclipse - `Help | Eclipse Marketplace -> 输入Checkstyle -> 安装Checkstyle Plugin-in`
        * 配置Checkstyle
            * `Window | Preferences | Checkstyle`
            * `项目右键 | Properties | Checkstyle -> Checkstyle for this project`
        * 执行Checkstyle : `项目/文件 右键 -> Checkstyle`
        * Checkstyle视窗 : `Window | Show View | Other | Checkstyle ...`

* 执行Checkstyle
  * [Ant Task](http://checkstyle.sourceforge.net/anttask.html)
  * [Command Line](http://checkstyle.sourceforge.net/cmdline.html)
    ```
    java -D<property>=<value>  \
         com.puppycrawl.tools.checkstyle.Main \
         -c <configurationFile> \
         [-f <format>] [-p <propertiesFile>] [-o <file>] \
         [-s <line:column>] [-gxs | --generate-xpath-suppression] [-tabWidth <length>] \
         [-t | --tree] [-T | --treeWithComments] [-J | treeWithJavadoc] [-j | --javadocTree] [-v] \
         file...
    ```
  * 使用下面提到的`Apache Maven Checkstyle Plugin`

* 查看之前版本的文档

    访问URL格式： `http://checkstyle.sourceforge.net/version/X.X`。`X.X`为Checkstyle版本号。例子：
    ` http://checkstyle.sourceforge.net/version/6.18`即为访问`6.18`版本的Checkstyle文档。

* 使用Checkstyle显示Java文件`AST(Abstract Syntax Tree)`结构，[参考文档](http://checkstyle.sourceforge.net/writingchecks.html#Tool_to_print_Java_tree_structure)
    
    * 命令行显示
        * PLAIN JAVA: `java -jar checkstyle-X.XX-all.jar -t MyClass.java`
        * JAVA WITH COMMENTS: `java -jar checkstyle-X.XX-all.jar -T MyClass.java`
    * GUI操作：`java -cp checkstyle-${projectVersion}-all.jar com.puppycrawl.tools.checkstyle.gui.Main`
    * Eclipse可以安装[Checkstyle AST Eclipse Viewer](https://github.com/sevntu-checkstyle/checkstyle-ast-eclipse-viewer)插件
    
#### Apache Maven Checkstyle Plugin
    
`maven-checkstyle-plugin:3.0.0`默认使用`Checkstyle 6.18`，依赖于JDK7，但可以自定义Checkstyle版本。

[使用文档 - 官网](http://maven.apache.org/components/plugins/maven-checkstyle-plugin/usage.html)

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
        <version>3.0.0</version>
        <executions>
          <execution>
            <id>checkstyle</id>
            <!-- 可以绑定到任意阶段 -->
            <phase>verify</phase>
            <goals>
              <!-- 可选目标：check、checkstyle、checkstyle-aggregate 。请参考文档-->
              <goal>checkstyle</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <!-- 可配置私有xml，参数类型：URL、File、build classpath resource -->
          <configLocation>google_checks.xml</configLocation>
        </configuration>
        <dependencies>
          <!-- 自定义插件依赖checkstyle的版本号 -->
          <dependency>
            <groupId>com.puppycrawl.tools</groupId>
            <artifactId>checkstyle</artifactId>
            <version>8.14</version>
          </dependency>
        </dependencies>
      </plugin>
    </plugins>
  </build>
  ...
  <!-- To use the report goals in your POM or parent POM -->
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
        <version>3.0.0</version>
          <reportSets>
            <reportSet>
              <reports>
                <report>checkstyle</report>
              </reports>
            </reportSet>
          </reportSets>
      </plugin>
    </plugins>
  </reporting>
</project>
```

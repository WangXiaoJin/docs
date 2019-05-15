## PMD

#### PMD

* 文档
    * [官网](https://pmd.github.io/)
    * [官方文档](https://pmd.github.io/pmd-6.9.0/index.html)

* 安装
    * 依赖于JRE 1.7或更高版本
    * 下载PMD,PMD安装包中已包含`PMD(Project Manager Design)`/`CPD(Copy-Paste Detector)`：<https://github.com/pmd/pmd/releases>
    * 使用OpenJDK或Java 11执行`Designer (./run.sh designer)`时，需要额外安装`OpenJFX`，`JAVAFX_HOME`环境变量指向其解压目录
    * 解压PMD目录，直接执行bin下面的命令

#### CLI命令行模式运行
    
PMD提供了很多工具，如`cpd`/`pmd`/`designer`/`bgastviewer`/`cpdgui`。Unix环境下，你可以使用`run.sh`来运行前面提到的
工具，如:`run.sh pmd ...`，后面的省略号即为pmd需要的参数。Window环境下，直接运行对应的脚本，如：`run.sh pmd`，`run.sh pmd`。

> 注：查看命令的帮助文档，后面加`-help`：`./run.sh pmd -help`

* 运行PMD
    
    两个比传参数：
    
    * `-d <path>`：需要解析的资源。可以为文件名、路径、Jar、Zip。
    * `-R <path>`：指定你需要使用的`ruleset`。PMD的Rulesets配置文件为XML格式。你可以通过引用`category`和`name`运行单个
    规则，[参考文档](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_making_rulesets.html#referencing-a-single-rule)
    如：`-R category/java/codestyle.xml/UnnecessaryModifier`，此为PMD内置规则，[内置规则文档](https://pmd.github.io/pmd-6.9.0/tag_rule_references.html)。
        > 注：所有的规则及默认提供的规则集都可在对应的jar包（或Github源码）里找到，如：`lib/pmd-java-6.9.0.jar`。
        `rulesets/java/basic.xml`已废弃，如想快速使用内置的规则集请用`rulesets/java/quickstart.xml`。  
    
    下面的可选配置项也会经常使用到：
    
    * `-f <format>`：生成报告的格式。默认为`text`格式，常用`text`/`xml`格式。可选格式：[参考文档](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_cli_reference.html#available-report-formats)
    * `-auxclasspath <classpath>`：Specifies the classpath for libraries used by the source code.配置此参数后可以让PMD
    通过反射更深入的解析source code，例如扫描[MissingOverride](https://pmd.github.io/pmd-6.9.0/pmd_rules_java_bestpractices.html#missingoverride),
    接口定义不在source code，而在auxclasspath中。
    
    > 注：[PMD CLI Reference - 官网](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_cli_reference.html)
    
    例子：
    
    * Linux/Unix
        
        ```bash
        ~ $ cd ~/bin/pmd-bin-6.9.0/bin
        ~/.../bin $ ./run.sh pmd -d ../../../src/main/java/ -f text -R rulesets/java/basic.xml
          
          .../src/main/java/com/me/RuleSet.java:123  These nested if statements could be combined
          .../src/main/java/com/me/RuleSet.java:231  Useless parentheses.
          .../src/main/java/com/me/RuleSet.java:232  Useless parentheses.
          .../src/main/java/com/me/RuleSet.java:357  These nested if statements could be combined
          .../src/main/java/com/me/RuleSetWriter.java:66     Avoid empty catch blocks
        ```
    * Windows
        
        ```bash
        C:\ > cd C:\pmd-bin-6.9.0\bin
        C:\...\bin > .\pmd.bat -d ..\..\src\main\java\ -f text -R rulesets/java/basic.xml
              
          .../src/main/java/com/me/RuleSet.java:123  These nested if statements could be combined
          .../src/main/java/com/me/RuleSet.java:231  Useless parentheses.
          .../src/main/java/com/me/RuleSet.java:232  Useless parentheses.
          .../src/main/java/com/me/RuleSet.java:357  These nested if statements could be combined
          .../src/main/java/com/me/RuleSetWriter.java:66     Avoid empty catch blocks
        ```

* 运行CPD
    
    CPD支持Java, JSP, C, C++, C#, Fortran and PHP源码，[支持的语言](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_cpd#supported-languages)
    
    两个必传参数：
    
    * `--files <path>`：需要解析的资源。可以为文件名、路径、Jar、Zip。
    * `--minimum-tokens <number>`：认为重复代码的最小token长度
    
    > 注：[CPD文档](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_cpd.html)
    
    例子：
    
    * Linux/Unix
        ```
        ~ $ cd ~/bin/pmd-bin-6.9.0/bin
        ~/.../bin $ ./run.sh cpd --minimum-tokens 100 --files /home/me/src
        
          Found a 7 line (110 tokens) duplication in the following files:
          Starting at line 579 of /home/me/src/test/java/foo/FooTypeTest.java
          Starting at line 586 of /home/me/src/test/java/foo/FooTypeTest.java
        
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
        ```
    * Windows
        ```
        C:\ > cd C:\pmd-bin-6.9.0\bin
        C:\...\bin > .\cpd.bat --minimum-tokens 100 --files c:\temp\src
        
          Found a 7 line (110 tokens) duplication in the following files:
          Starting at line 579 of c:\temp\src\test\java\foo\FooTypeTest.java
          Starting at line 586 of c:\temp\src\test\java\foo\FooTypeTest.java
        
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
                  assertEquals(Boolean.TYPE, expressions.get(index++).getType());
        ```

* 文档

    * [Making rulesets - 重要](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_making_rulesets.html)
    * [Configuring rules - 重要](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_configuring_rules.html)
    * [Suppressing PMD warnings - 重要](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_suppressing_warnings.html)
    * [Suppressing CPD warnings - 重要](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_cpd.html#suppression)
    * [Incremental Analysis - 重要](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_incremental_analysis.html)
    * [Writing a custom rule](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_extending_writing_pmd_rules.html)
    * [Writing XPath rules](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_extending_writing_xpath_rules.html)
    * [Java Rules - 重要](https://pmd.github.io/pmd-6.9.0/pmd_rules_java_bestpractices.html)
    * [Rules Priority - 重要](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_extending_rule_guidelines.html#how-to-define-rules-priority)
    * [Java code metrics](https://pmd.github.io/pmd-6.9.0/pmd_java_metrics_index.html)
    * [How PMD Works](https://pmd.github.io/pmd-6.9.0/pmd_devdocs_how_pmd_works.html)
    
#### Maven PMD Plugin

* [PMD官网](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_tools_maven.html)
* [Apache插件官网](http://maven.apache.org/plugins/maven-pmd-plugin/index.html)

#### IDE集成

* [Eclipse、IDEA等IDE集成PMD](https://pmd.github.io/pmd-6.9.0/pmd_userdocs_tools.html#idea)

#### PMD类似项目

* [PMD类似项目集合](https://pmd.github.io/pmd-6.9.0/pmd_projectdocs_trivia_similarprojects.html)



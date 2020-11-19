## SonarQube

#### 安装SonarQube

1. 安装先决条件 - [官方文档](https://docs.sonarqube.org/display/SONAR/Requirements)
    
    * 服务器必须安装`Oracle JRE 8 or OpenJDK 8`
    * 在`Mac OS X`系统中推荐安装`Oracle JDK 8`，而不是`Oracle JRE`。因为`Oracle JRE`的Java环境变量没有配置完整。[链接](https://stackoverflow.com/questions/15624667/mac-osx-java-terminal-version-incorrect)
    * SonarQube最少需要2GB内存，及预留1GB内存给系统使用
    * 数据库选用Mysql时，请使用`5.6`、`5.7`版本
    * Linux系统配置
    
        * vm.max_map_count is greater or equals to 262144
        * fs.file-max is greater or equals to 65536
        * the user running SonarQube can open at least 65536 file descriptors
        * the user running SonarQube can open at least 2048 threads
        
            你可以通过下面的命令查看配置：
            ```bash
            shell> sysctl vm.max_map_count
            shell> sysctl fs.file-max
            shell> ulimit -n
            shell> ulimit -u
            ```
            
            以**root**用户通过以下命令动态修改当前session配置：
            ```bash
            shell> sysctl -w vm.max_map_count=262144
            shell> sysctl -w fs.file-max=65536
            shell> ulimit -n 65536
            shell> ulimit -u 2048
            ```
            
            如果想永久改变这些配置项，可以修改`/etc/sysctl.d/99-sonar.conf`（或`/etc/sysctl.conf`）、
            `/etc/security/limits.d/99-sonar.conf`（或`/etc/security/limits.conf`）文件，具体配置略。  
            
            以下SonarQube以`sonar`作为启动用户为例，配置如下：
            ```
            sonar   -   nofile   65536
            sonar   -   nproc    2048
            ```
        
        * seccomp filter
        
            默认情况下Elasticsearch 使用seccomp filter。在多数linux版本中`seccomp filter`已开启，但在某些版本中
            （如：Red Hat Linux 6）没有此特性，此时你就需要在`sonar.properties`配置文件中修改`sonar.search.javaAdditionalOpts`
            配置项，禁用`seccomp filter`功能：
            
            ```
            sonar.search.javaAdditionalOpts=-Dbootstrap.system_call_filter=false
            ```
            
            检查seccomp 在你的系统内核中是否可用：
            ```bash
            shell> grep SECCOMP /boot/config-$(uname -r)
            ```
            如果你的内核存在seccomp，你将会看到：
            ```
            CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
            CONFIG_SECCOMP_FILTER=y
            CONFIG_SECCOMP=y
            ```

2. 安装

    * 安装sonarqube
        ```bash
        shell> wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-7.3.zip
        shell> unzip sonarqube-7.3.zip -d /usr/local
        shell> cd /usr/local && mv sonarqube-7.3 sonar
        shell> useradd sonar -s /sbin/nologin
        shell> chown -R sonar:sonar sonar
        ```
    
    * 安装Mysql - [参考文档](../mysql/MySQL%20Server、MySQL%20Utilities安装及使用.md)，创建user、database：
        ```bash
        shell> mysql -u root -p
        Enter password: (enter root password here)
        mysql> CREATE USER 'sonar'@'%' IDENTIFIED BY 'self-password';
        mysql> CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
        mysql> GRANT ALL PRIVILEGES ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'self-password' WITH GRANT OPTION;
        mysql> FLUSH PRIVILEGES;
        ```
    
    * 修改`<install_directory>/conf/sonar.properties`配置文件
    
        ```
        # 修改下面配置项
        sonar.jdbc.username=sonar
        sonar.jdbc.password=self-password
        # Only InnoDB storage engine is supported
        sonar.jdbc.url=jdbc:mysql://192.168.1.101:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
        sonar.web.javaOpts=-Xmx2g -Xms2g -server -XX:+HeapDumpOnOutOfMemoryError
        sonar.ce.javaOpts=-Xmx512m -Xms512m -server -XX:+HeapDumpOnOutOfMemoryError
        sonar.search.javaOpts=-Xms2g -Xmx2g -server -XX:+HeapDumpOnOutOfMemoryError
        # 配置了此参数后，访问web服务接口不需要提供用户名及密码。直接在HTTP请求头增加X-Sonar-Passcode配置项就能访问。
        # 如你想加密此配置，参考文档：https://redirect.sonarsource.com/doc/settings-encryption.html
        sonar.web.systemPasscode=sonar
        #相对路径或绝对路径，启动sonarqube的用户需要data、temp文件夹的写权限
        #sonar.path.data=data
        #sonar.path.temp=temp
        ```
        
        你还可以修改其他配置项：数据库连接池；JVM参数；绑定IP；Web配置；SSO；LDAP；日志；
        
        默认日志路径：
        * Main process (aka. App) logs in logs/sonar.log
        * Web Server (aka. Web) logs in logs/web.log
        * Compute Engine (aka. CE) logs in logs/ce.log
        * Elasticsearch (aka. ES) logs in logs/es.log
  
        > 配置文件可以使用环境变量：sonar.jdbc.url= ${env:SONAR_JDBC_URL}
    
    * Securing the Server Behind a Proxy
        
        To run the SonarQube server over HTTPS, you must build a standard reverse proxy infrastructure.
        
        The reverse proxy must be configured to set the value "X_FORWARDED_PROTO: https" in each HTTP request header. Without this property, redirection initiated by the SonarQube server will fall back on HTTP.
    
    * 修改sonar运行时用户
    
        修改对应`sonar.sh`文件（例：`$SONAR_HOME/bin/linux-x86-64/sonar.sh`）的`RUN_AS_USER`配置：
        ```bash
        RUN_AS_USER=sonar
        ```
    
    * 启动服务
        
        ```bash
        shell> $SONAR_HOME/bin/linux-x86-64/sonar.sh start
        ```
        项目启动成功后，访问<http://localhost:9000>页面，使用`账号：admin 密码：admin`登录系统
        
        > 注：如果本地防火墙启动，且拦截了127.0.0.1时，则sonar启动不了。任何友好提示信息都没有，只有jvm timeout信息。
        
    * Running SonarQube as a Service（可选）
    
        创建`/etc/init.d/sonar`文件：
        ```bash
        #!/bin/sh
        #
        # rc file for SonarQube
        #
        # chkconfig: 345 96 10
        # description: SonarQube system (www.sonarsource.org)
        #
        ### BEGIN INIT INFO
        # Provides: sonar
        # Required-Start: $network
        # Required-Stop: $network
        # Default-Start: 3 4 5
        # Default-Stop: 0 1 2 6
        # Short-Description: SonarQube system (www.sonarsource.org)
        # Description: SonarQube system (www.sonarsource.org)
        ### END INIT INFO
         
        /usr/bin/sonar $*
        ```
        
        开机启动 (Ubuntu, 32 bit):
        ```bash
        shell> sudo ln -s $SONAR_HOME/bin/linux-x86-32/sonar.sh /usr/bin/sonar
        shell> sudo chmod 755 /etc/init.d/sonar
        shell> sudo update-rc.d sonar defaults
        ```
        
        开机启动(RedHat, CentOS, 64 bit):
        ```bash
        shell> sudo ln -s $SONAR_HOME/bin/linux-x86-64/sonar.sh /usr/bin/sonar
        shell> sudo chmod 755 /etc/init.d/sonar
        shell> sudo chkconfig --add sonar
        ```
    
    * Monitoring
        
        CPU and RAM usage on each node have to be monitored separately with an APM. 
        
        In addition, we provide a Web API `api/system/health` you can use to validate all of the nodes of your cluster are operation.
        
        * GREEN: SonarQube is fully operational  
        * YELLOW: SonarQube is usable, but it needs attention in order to be fully operational
        * RED: SonarQube is not operational
        
        To call it from monitoring system without having to give admin credentials, it is possible to setup a System 
        Passcode through the property `sonar.web.systemPasscode`. This has to be configured in the `sonar.properties`.
        
        * [Java Process Memory](https://docs.sonarqube.org/display/SONAR/Java+Process+Memory)
        * [JMX MBeans - 【重要】](https://docs.sonarqube.org/display/SONAR/JMX+MBeans)
        * [System Info（系统信息及日志下载） - 【重要】](https://docs.sonarqube.org/display/SONAR/System+Info)
    
    * 安装插件的两种方式：
    
        * 通过SonarQube UI界面自动安装
        
            进入`Administration > Marketplace`页面，找到你想安装的插件。点击安装按钮，待下载完成后，点击`Restart`按钮重启实例。
        
        * 手动安装
        
            下载插件的jar包，将下载的jar包放置`$SONARQUBE_HOME/extensions/plugins`目录下，删除已存在的插件版本。重启SonarQube服务。
        
        [插件列表](https://docs.sonarqube.org/display/PLUG)
        
        [Marketplace](https://docs.sonarqube.org/display/SONAR/Marketplace)
        
        > 注：安装商业版的插件，需要在`Administration > Configuration > License Manager`里设置许可密钥。
        
3. UI界面配置 - **以下配置根据自己的需求而定**
    
    * 配置项目默认为私有(Private)项目：
        
        `Administration -> Projects -> Management -> Default visibility of new projects -> 选择Private`
    
    * 禁止`匿名用户`访问SonarQube UI和特定的Web API：
    
        `Administration -> Configuration -> General Settings -> Security -> Force user authentication（开启）`
        
        或者也可在Sonar Web配置文件中增加此项：`sonar.forceAuthentication=true`
    
    * 配置`Default template`【禁止`sonar-users组（任意用户）`/允许`sonar-administrators`】访问`project`及项目源码：
        
        `Administration -> Security -> Permission Templates -> Default template`
        
        * `sonar-administrators` ：`勾选` `Browse` 、 `See Source Code`
        * `sonar-users` ：`反选` `Browse` 、 `See Source Code`
    
    * 禁止`Anyone`（任意用户）创建project，只分配权限给`sonar-administrators`
        
        `Administration -> Security -> Global Permissions`
        
        * `Anyone` ：`反选` `Create Projects`
        * `sonar-administrators` ：`勾选` `Create Projects`
        
    * 配置Sonar服务的域名地址，用于Webhook及邮件内容中指定Sonar服务地址
        
        `Administration -> Configuration -> General Settings -> General -> Server base URL`
        
        * 配置当前服务的域名地址，如：`http://192.168.1.100:9000`
    
4. Web API

    * 直接访问地址：<http://localhost:9000/web_api>
    * 直接点击Sonar UI底部的`Web API`链接
    * 访问`http://localhost:9000/api`，返回的是JSON数据结构
    
    **`Web API`需要身份认证及授权，通过下面两种方式认证：**
    
    * `User Token` - This is the recommended way. Benefits are described in the page [User Token](https://docs.sonarqube.org/display/SONAR/User+Token). 
    Token is sent via the login field of HTTP basic authentication, without any password.
        
        ```bash
        # note that the colon after the token is required in curl to set an empty password 
        shell> curl -u THIS_IS_MY_TOKEN: https://sonarqube.com/api/user_tokens/search
        ```
    
    * `HTTP Basic Access` - Login and password are sent via the standard HTTP Basic fields:
    
        ```bash
        shell> curl -u MY_LOGIN:MY_PASSWORD https://sonarqube.com/api/user_tokens/search
        ```
    
5. Sonar集成SpotBugs, FindSecBugs, PMD, Checkstyle的两种方式
  
    1. 已插件形式安装PMD、Checkstyle等，参考上面的文档。在SonarQube7.3版本不支持`sonar-checkstyle:4.11`，
    因为其还在使用Sonar已废弃的API。[Sonar各版本兼容插件](https://docs.sonarqube.org/display/PLUG/Plugin+Version+Matrix)
    2. [直接导入SpotBugs, FindSecBugs, PMD, Checkstyle Issues Reports至Sonar](https://docs.sonarqube.org/display/PLUG/Importing+SpotBugs%2C+FindSecBugs%2C+PMD%2C+Checkstyle+Issues+Reports)

5. 文档

    * [Benchmark](https://docs.sonarqube.org/display/SONAR/Benchmark)
    * [Concepts -【重要】](https://docs.sonarqube.org/display/SONAR/Concepts)
    * Issue分配给指定用户
        * [Automatic](https://docs.sonarqube.org/display/SONAR/Automatic+Issue+Assignment)
        * [Manual](https://docs.sonarqube.org/display/SONAR/Authorization#Authorization-User)
    * [Issues概念及相关术语 - 【Blocker/Critical/Major/Minor/Info】 -【重要】](https://docs.sonarqube.org/display/SONAR/Issues)
    * Security - 重要
        * Security Hotspots - Security Hotspots are a special type of issue that identify sensitive areas of code that 
        should be reviewed by a Security Auditor to determine if they are truly Vulnerabilities.  
        See Security Audits and Reports for detail on Security Hotspots and the audit process.
        * [Security Hotspots / Security Auditor / Security Reports](https://docs.sonarqube.org/display/SONAR/Security+Audit+and+Reports)
        * [Security-related rules / CWE / SANS Top 25 / OWASP Top 10](https://docs.sonarqube.org/display/SONAR/Security-related+rules)
    * [Rules Types and Severities定义 - 【Bugs/Vulnerabilities/Security Hotspots/Code Smells】](https://docs.sonarqube.org/display/SONAR/Rules+Types+and+Severities)
        * Code Smell (Maintainability domain)
        * Bug (Reliability domain)
        * Vulnerability (Security domain)
        * Security Hotspot (Security domain)
    * [Metric Definitions -【重要】](https://docs.sonarqube.org/display/SONAR/Metric+Definitions)
    * [指定解析范围](https://docs.sonarqube.org/display/SONAR/Narrowing+the+Focus)
    * [Webhooks -【重要】](https://docs.sonarqube.org/display/SONAR/Webhooks)
    * [Adding Hooks](https://docs.sonarqube.org/display/DEV/Adding+Hooks)
    * [Custom Measures](https://docs.sonarqube.org/display/SONAR/Custom+Measures)
    * [快照（分析报告）删除策略](https://docs.sonarqube.org/display/SONAR/Housekeeping)
    * [Authentication](https://docs.sonarqube.org/display/SONAR/Authentication)
    * [Authorization](https://docs.sonarqube.org/display/SONAR/Authorization)
        * **Anyone** is a group that exists in the system, but that cannot be managed. Every user belongs to this group, including Anonymous user.
        * **sonar-users** is the default group to which users are automatically added.
    * [LDAP Integration](https://docs.sonarqube.org/display/SONAR/LDAP+Integration)
    * [Settings Encryption（加密配置项）-【重要】](https://docs.sonarqube.org/display/SONAR/Settings+Encryption)
    * [各插件配置及文档-【重要】](https://docs.sonarqube.org/display/PLUG)
    * Developing a Plugin
        * [Build Plugin](https://docs.sonarqube.org/display/DEV/Build+Plugin)
        * [API Basics](https://docs.sonarqube.org/display/DEV/API+Basics)
    * Custom Rules - 依赖于上面的`Developing a Plugin`
        * [Custom Rules Examples - Cobol/Java/JS/Php/Rpg](https://github.com/SonarSource/sonar-custom-rules-examples)
        * Custom Rules for Java -【重要】
            * [Custom Rules for Java](https://docs.sonarqube.org/display/PLUG/Custom+Rules+for+Java)
            * [Writing Custom Java Rules 101](https://docs.sonarqube.org/display/PLUG/Writing+Custom+Java+Rules+101)
        * [Custom Rules for SonarJS](https://docs.sonarqube.org/display/PLUG/Custom+Rules+for+SonarJS)
    * [Adding Coding Rules](https://docs.sonarqube.org/display/DEV/Adding+Coding+Rules)
        * [Adding Coding Rules using `XPath`](https://docs.sonarqube.org/display/DEV/Adding+Coding+Rules+using+XPath)
        * [Adding Coding Rules using `Java`](https://docs.sonarqube.org/display/DEV/Adding+Coding+Rules+using+Java)
    * [Extending Web Application](https://docs.sonarqube.org/display/DEV/Extending+Web+Application)
    
6. FAQ

    * 误操作导致自己没有管理员权限
    
        重新关联管理员角色：
        ```
        INSERT INTO user_roles(user_id, role) VALUES ((select id from users where login='mylogin'), 'admin');
        ```
    
    * 重置admin密码为`admin`
    
        重新关联管理员角色：
        ```
        update users set crypted_password = '$2a$12$uCkkXmhW5ThVK8mpBvnXOOJRLd64LJeHTeCkSuB3lfaR2N0AYBaSi', salt=null, hash_method='BCRYPT' where login = 'admin'
        ```
    
    * How to trigger a full SonarQube ES reindex?
    
        Currently, the only way to force a reindex is to:
        * Stop your server
        * Remove the contents of the `$SQ_HOME/data/es5` directory
        * Start your server
    
    * I can't use my HTTP Proxy since I upgraded to Java8u111：[参考链接](http://www.oracle.com/technetwork/java/javase/8u111-relnotes-3124969.html)
    
        If you are getting this error in the logs when trying to use the Marketplace:
        ```
        java.io.IOException: Unable to tunnel through proxy. Proxy returns "HTTP/1.1 407 Proxy Authentication Required
        ```
        
        you probably upgraded your Java8 installation with an update greater than 111. To fix that, change your sonar.properties like this:
        ```
        sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Djdk.http.auth.tunneling.disabledSchemes=""
        ```
        
    * How to remove False-Positive issues?
        
        **False-Positive and Won't Fix**
        
        You can mark individual issues as False Positive or Won't Fix through the issues interface. However, this solution 
        doesn't work across branches - you'll have to re-mark the issue False Positive for each branch under analysis. 
        So an in-code approach may be preferable if multiple branches of a project are under analysis:
        
        **//NOSONAR**
        
        You can use the generic mechanism implemented in SonarQube: put `//NOSONAR` at the end of the line of the issue. 
        This will suppress the all issues - now and in the future - that might be raised on the line.
        
        **`@SuppressWarnings` for Java**
        
        The //NOSONAR tag is useful to deactivate all rules at a given line but is not suitable to deactivate all rules 
        (or only a given rule) for all the lines of a method or a class. This is why support for `@SuppressWarnings("all")`
        has been added to SonarQube.
        
        `SINCE 2.8` of Java Plugin, you can also use @SuppressWarnings annotation with a list of rule keys: 
        `@SuppressWarnings("squid:S2078")` or  `@SuppressWarnings({"squid:S2078", "squid:S2076"})`. 
            
        * For v2.8 : it will only work with rules provided by SonarSource, so rules having a key starting with squid. 
        * For v2.9+ : the limitation is gone and it will work with others java rule engines
        * For v3.4 and before: Annotation is supported only at Class and Method level
        * For v3.5+: Annotation is also supported at Field / Local Variable / Method Parameter level
        * Known limitation: Annotation can not be used to suppress issues occuring at file level
        
        **Switch Off Issues**
        
        You can review an issue to flag it as false positive directly from the user interface.

    * Analysis errors out with java.lang.OutOfMemoryError: GC overhead limit exceeded. What do I do?
    
        This means that your project is too large or too intricate for the scanner to analyze with the default memory allocation. 
        To fix this you'll want to allocate a larger heap (using -Xmx[numeric value here]) to the process running the analysis. 
        Some CI engines may give you an input to specify the necessary values, for instance if you're using 
        a Maven Build Step in a Jenkins job to run analysis. Otherwise, use Java Options to set a higher value. 
        Note that details of setting Java Options are omitted here because they vary depending on the environment.

    * Unexpected EOF read on the socket
        * [SonarScanner MSBuild: Socket Error while uploading report to server](https://community.sonarsource.com/t/sonarscanner-msbuild-socket-error-while-uploading-report-to-server/2091)

    * 设置`sonar.ws.timeout`：[文档](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)

    * `Null pointers should not be dereferenced`(`java:S2259`) 误报解决方案 - [Overriding a java rule with a custom version](https://community.sonarsource.com/t/overriding-a-java-rule-with-a-custom-version/623)



#### SonarQube Scanners

* SonarQube Scanner`命令行模式`

    下载页面：https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner
    
    ```bash
    shell> wget https://sonarsource.bintray.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.2.0.1227-linux.zip
    shell> unzip sonar-scanner-cli-3.2.0.1227-linux.zip
    shell> mv sonar-scanner-3.2.0.1227-linux sonar-scanner
    # sonar-scanner查看帮助信息
    shell> sonar-scanner/bin/sonar-scanner -h
        usage: sonar-scanner [options]
        Options:
         -D,--define <arg>     Define property
         -h,--help             Display help information
         -v,--version          Display version information
         -X,--debug            Produce execution debug output
    ```
    
    > 注：sonar-scanner可通过`-D`来传递各种参数。如想得到更多的debug信息：`sonar-scanner -Dsonar.verbose=true`  
    更全面的参数配置请参考：[官方文档 - Analysis Parameters - 【重要】](https://docs.sonarqube.org/display/SONAR/Analysis+Parameters)
    
    **修改`<install_directory>/conf/sonar-scanner.properties`配置**
    
    ```
    # 修改成自己的sonar server地址
    sonar.host.url=http://localhost:9000
    ```
    
    **执行sonar-scanner**
    
    任何用户只要有`Execute Analysis`权限，就可以执行sonar-scanner并上传报告。（Administration --> Security --> Global Permissions/Permission Templates --> Execute Analysis）。
    
    如果`Anyone`组没有分配`Execute Analysis`权限或SonarQube服务的` sonar.forceAuthentication`配置为`true`，则必须配置`sonar.login`配置。
    例：`sonar-scanner -Dsonar.login=[my analysis token]` ，可用账号/密码 或 Token两种格式。[User Token](https://docs.sonarqube.org/display/SONAR/User+Token)
    
    1. 使用配置文件运行。在项目的根目录下配置`sonar-project.properties`：
        
        ```
        # must be unique in a given SonarQube instance
        sonar.projectKey=com.test:project
        # this is the name and version displayed in the SonarQube UI. Was mandatory prior to SonarQube 6.1.
        sonar.projectName=My project
        sonar.projectVersion=1.0
         
        # Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
        # This property is optional if sonar.modules is set. 
        sonar.sources=.
         
        # Encoding of the source code. Default is default system encoding
        sonar.sourceEncoding=UTF-8
        ```
    
    2. 命令行传参
        
        ```bash
        shell> sonar-scanner -Dsonar.projectKey=myproject -Dsonar.sources=src1
        ```
    
    > 注：[项目多模块结构、命令行配置](https://docs.sonarqube.org/display/SCAN/Advanced+SonarQube+Scanner+Usages)
    
    > 注：执行SonarQube Scanner报错：`Please provide compiled classes of your project with sonar.java.binaries property` 参考[官方文档](https://docs.sonarqube.org/display/PLUG/SonarJava)
    
    > 注：执行SonarQube Scanner时**增加自定义数据**，用于WebHook：`sonar-scanner -Dsonar.analysis.scmRevision=628f5175ada0d685fd7164baa7c6382c1f25cab4 -Dsonar.analysis.buildNumber=12345`
    
* Analyzing with SonarQube Scanner for `Maven`【重要】 : [官方文档](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven)
    * 输出DEBUG日志
        
        Regarding debug logs this is an expected behavior. We are now writing logs in the Maven logger (no more directly to System.out). 
        It brings a lot of good side effects (better integration, support of log colorizer) but on the other side if you don't enable Maven DEBUG logs you won't see SQ DEBUG logs.
        
        * `mvn sonar:sonar -Dsonar.verbose=true` => SQ write DEBUG logs but Maven doesn't display them
        * `mvn sonar:sonar -Dsonar.verbose=true -X` => works fine
        * `mvn sonar:sonar -X` => works fine since we are automatically enabling SQ DEBUG logs when -X is used.
        
        
* Analyzing with SonarQube Scanner for `Gradle`【重要】 : [官方文档](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Gradle)
* Analyzing with SonarQube Scanner for `Jenkins`【重要】 : [官方文档](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins)
* Analyzing with SonarQube Scanner for `Ant` : [官方文档](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Ant)
* Analyzing with SonarQube Scanner for `MSBuild` : [官方文档](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild)
* Analyzing with SonarQube Scanner for `VSTS/TFS` : [官方文档](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Extension+for+VSTS-TFS)

* FAQ

    * Java heap space error or java.lang.OutOfMemoryError
        
        Increase the memory via the SONAR_SCANNER_OPTS environment variable:
        
        ```
        export SONAR_SCANNER_OPTS="-Xmx512m"
        ```
        
        On Windows environments, avoid the double-quotes, since they get misinterpreted and combine the two parameters into a single one.
        
        ```
        set SONAR_SCANNER_OPTS=-Xmx512m
        ```

    * Unsupported major.minor version
        
        Upgrade the version of Java being used for analysis or use one of the native package (that embed its own Java runtime). SonarQube 5.6+ requires Java 8.

    * 在解析已集成`SCM`的项目时，如果报没有权限操作`SCM`，则可配置如下参数
        
        SVN：可在执行命令时传参数或在`UI->Administration->SCM`页面配置`sonar.svn.username`/`sonar.svn.password.secured`，[参考源码](https://github.com/SonarSource/sonar-scm-svn/blob/462d14c36f4d2c9fdd8847a95aaf04538f062f0c/sonar-scm-svn-plugin/src/main/java/org/sonar/plugins/scm/svn/SvnConfiguration.java)
        
        集成 `SVN SCM`错误内容：Failed to execute goal org.sonarsource.scanner.maven:sonar-maven-plugin:3.4.1.1168:sonar (default-cli) on project xxxx: 
        Error when executing blame for file src/main/java/...: svn: E170001: Authentication required for '<http://xxx.com:80> Repository' -> [Help 1]
    
* 文档
    
    * [Analysis Parameters](https://docs.sonarqube.org/display/SONAR/Analysis+Parameters)
    * [CoverageReport/TestExecutionReport](https://docs.sonarqube.org/display/SONAR/Generic+Test+Data)
    * [ExternalIssuesReport](https://docs.sonarqube.org/display/SONAR/Generic+Issue+Data)
    * [sonar-scanning-examples（提供的项目样本）](https://github.com/SonarSource/sonar-scanning-examples)


#### SonarLint 

* [`IntelliJ IDEA`使用`SonarLint`插件文档](https://www.sonarlint.org/intellij/)
* [`Eclipse`使用`SonarLint`插件文档](https://www.sonarlint.org/eclipse/)
* [`Visual Studio`使用`SonarLint`插件文档](https://www.sonarlint.org/visualstudio/)
* [`VS Code`使用`SonarLint`插件文档](https://www.sonarlint.org/vscode/)
* [`Atom`使用`SonarLint`插件文档](https://www.sonarlint.org/atom/)

* [`Eclipse` 在线/离线 安装`SonarLint`](https://blog.csdn.net/limm33/article/details/51166840)



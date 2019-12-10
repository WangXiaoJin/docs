# NexusRepository安装及使用

## 安装

Nexus3 [下载链接](https://help.sonatype.com/repomanager3/download)

### 1. System Requirements

* Dedicated Operating System User Account

    The NXRM process user is typically named `nexus` and must be able to create a valid shell，do not run Nexus Repository Manager 3 as the `root` user.

* Adequate File Handle Limits
    
    NXRM3 will most likely want to consume more file handles than the per user default value allowed by your Linux or OSX operating system.

    Running out of file descriptors can be disastrous and will most probably lead to data loss. Make sure to increase the limit on the number of open files descriptors for the user running Nexus Repository Manager `permanently to 65,536 or higher` prior to starting.
    
    以下只提Linux系统配置，如安装至其他系统，则参考[官网配置](https://help.sonatype.com/repomanager3/system-requirements)。
    
    To set the maximum number of open files for both soft and hard limits for the `nexus` user to `65536`, add the following line to the `/etc/security/limits.conf` file：
    ```
    nexus - nofile 65536
    ```
    
    On `Ubuntu systems` there is a caveat: Ubuntu ignores the `/etc/security/limits.conf` file for processes started by `init.d`.

    So if NXRM is started using init.d there, edit `/etc/pam.d/common-session` and uncomment the following line ( remove the hash # and space at the beginning of the line):
    ```
    # session    required   pam_limits.so
    ```
    
    If you're using `systemd` to launch the server the above won't work. Instead, modify the configuration file to add a `LimitNOFILE` line:
    ```
    [Unit]
    Description=nexus service
    After=network.target
    
    [Service]
    Type=forking
    LimitNOFILE=65536
    ExecStart=/opt/nexus/bin/nexus start
    ExecStop=/opt/nexus/bin/nexus stop
    User=nexus
    Restart=on-abort
    
    [Install]
    WantedBy=multi-user.target
    ```

* Java
    
    Nexus Repository Manager 3 is a Java server application that requires a `Java 8` SE standard compatible runtime.
    
    **Java runtime versions other than 8 are not supported - do not use them.**
    
    **Only 64 bit Java is supported, do not use 32 bit Java.**

* CPU
    
    Performance is primarily bounded by IO (disk and network) rather than CPU. Available CPUs will impact longer running operations and also the thread allocation algorithms of the web container.
    
    **Minimum CPUs: 4**
    
    **Recommended CPUs: 8+**

* Memory
    
    **Configurable Memory Types**
    
    Visit the [Configuring the Runtime Enviroment](https://help.sonatype.com/repomanager3/installation/configuring-the-runtime-environment#ConfiguringtheRuntimeEnvironment-ConfiguringtheRuntimeEnvironment-ConfiguringMemory) page to learn how to change the default memory settings.

    **JVM Heap Memory**
    
    Heap memory stores runtime application objects. A min ( -Xms ) and max ( -Xmx ) value must be specified and the values should be identical.

    Increasing the heap memory larger than recommendations or setting the min and max values to be different is **not recommended**. This will create performance issues causing the operating system to thrash needlessly.

    Unless you have evidence that a max heap of 4GB is consistently utilized or there are frequent lengthy garbage collection pauses that cannot be explained by software bugs, then **do not set max heap size larger than 4GB**.
    
    **JVM Direct Memory**
    
    Direct memory is allocated outside of and distinct from heap memory. A max value must be configured.
    
    **Host Physical Memory**
    
    The total memory allocated to the entire operating system or virtual hardware, commonly referred to as RAM.
    
    **Memory Requirements**
    
    The requirements assume there are no other significant memory hungry processes running on the same host.
    
    |  | JVM Heap  | JVM Direct | Host Physical/RAM |
    | :-------: | :------: | :------: | :------: |
    | **Minimum ( default )**  | 2703MB | 2703MB | 8GB |
    | **Maximum** | 4GB | (host physical/RAM * 2/3) - JVM max heap | no limit |
    
    **General Memory Guidelines**
    
    * minimum physical/RAM memory on the host 8GB
    * minimum heap ( -Xms ) must equal set maximum heap ( -Xmx )
    * minimum heap size 2703MB
    * maximum heap size <= 4GB
    * minimum direct memory ( -XX:MaxDirectMemorySize ) size 2703MB
    * minimum unallocated host physical/RAM memory should be no less than 1/3 of total physical RAM to allow for virtual memory swap
    * `max heap` + `max direct memory` <= `host physical/RAM * 2/3`
    
    **Instance Memory Sizing Profiles**

    | Profile Use Case  | Physical/RAM Memory  |
    | :-------: | :------: |
    | small, personal <br><br> repositories < 20  <br> total blobstore size < 20GB <br> single repository format type |  8GB minimum | 
    | medium, team <br><br> repositories < 50  <br> total blobstore size < 200GB <br> a few repository formats |  16GB | 
    | large, enterprise <br><br> repositories > 50  <br> total blobstore size > 200GB <br> diverse set of repository formats  |  32GB+ | 
    
    **Example Maximum Memory Configurations**
    
    | Physical/RAM Memory  | Example Maximum Memory Configuration |
    | :-------: | :------: |
    | 8GB | -Xms2703M <br> -Xmx2703M <br> -XX:MaxDirectMemorySize=2703M |
    | 12GB | -Xms4G <br> -Xms4G <br> -XX:MaxDirectMemorySize=4014M |
    | 16GB | -Xms4G <br> -Xmx4G <br> -XX:MaxDirectMemorySize=6717M |
    | 32GB | -Xms4G <br> -Xmx4G <br> -XX:MaxDirectMemorySize=17530M |
    | 64GB | -Xms4G <br> -Xmx4G <br> -XX:MaxDirectMemorySize=39158M |
    
    **Advanced Database Memory Tuning - 【重要】**
    
    Refer to another article which outlines [additional memory tuning procedures](https://support.sonatype.com/hc/en-us/articles/115007093447?_ga=2.137310579.2134597265.1574671641-534816866.1574671641).
    
    **Temporary Directory**
    
    The [temporary directory](https://help.sonatype.com/repomanager3/installation/configuring-the-runtime-environment#ConfiguringtheRuntimeEnvironment-ConfiguringtheTemporaryDirectory`) at $data-dir/tmp` must not be mounted with `noexec` or repository manager startup will fail with `java.lang.UnsatisfiedLinkError`  message of  `failed to map segment from shared object: Operation not permitted` .
    
    **Disk IO**
    
    > [Optimizing Nexus Disk IO Performance](https://support.sonatype.com/hc/en-us/articles/213465258-Optimizing-Nexus-Disk-IO-Performance?_ga=2.138188403.2134597265.1574671641-534816866.1574671641) - Avoid Recording File Access Times On Reads
    
### 2. Install And Run

#### 安装及配置

```bash
# 解压出 "nexus-3.19.1-01" --> 应用程序文件夹 。"sonatype-work" --> 数据文件夹
shell> tar xvzf nexus-3.19.1-01-unix.tar.gz

# 添加nexus用户及用户组
shell> useradd nexus

# 修改 run_as_user="nexus" ，并去除前面的注释
shell> vim ./nexus-3.19.1-01/bin/nexus.rc

shell> chown -R nexus:nexus nexus-3.19.1-01
shell> chown -R nexus:nexus sonatype-work

# 后台运行Nexus，日志输出到日志文件。后台运行指令：
#   - start
#   - stop
#   - restart
#   - force-reload
#   - status
shell> ./nexus-3.19.1-01/bin/nexus start

# 在当前shell启动Nexus：日志会输出到控制台，按"CTRL-C"会关闭Nexus
shell> ./nexus-3.19.1-01/bin/nexus run
```

[Configuring the Runtime Environment](https://help.sonatype.com/repomanager3/installation/configuring-the-runtime-environment) - 文档内容：
* To change JVM memory
* Changing the HTTP Port
* Setup HTTPS
* Changing the Context Path
* Configuring the Data Directory
* Configuring the Temporary Directory

> 注：修改JVM参数 - `./nexus-3.19.1-01/bin/nexus.vmoptions`

> [Retry Limit Configuration](https://help.sonatype.com/repomanager3/configuration/retry-limit-configuration) - 可选配置

#### Run as a Service

可以通过`init.d`或`systemd`让Nexus以服务形式运行。

* `init.d`

    给`$installdir/bin/nexus`创建`/etc/init.d/nexus`链接
        
    ```bash
    # 安装目录根据实际情况而定
    shell> sudo ln -s /opt/nexus-3.15.2-01/bin/nexus /etc/init.d/nexus
    ```
    
    * `chkconfig`
        
        ```bash
        cd /etc/init.d
        # 添加 nexus 服务。chkconfig 管理 `/etc/rc[0-6].d` 目录下的链接
        sudo chkconfig --add nexus
        sudo chkconfig --levels 345 nexus on
        sudo service nexus start
        ```
    * `update-rc.d` - 和`chkconfig`功能类似
        
        ```bash
        cd /etc/init.d
        sudo update-rc.d nexus defaults
        sudo service nexus start
        ```

* `systemd`

    创建`/etc/systemd/system/nexus.service`，文件内容：
    ```
    [Unit]
    Description=nexus service
    After=network.target
      
    [Service]
    Type=forking
    LimitNOFILE=65536
    ExecStart=/opt/nexus-3.15.2-01/bin/nexus start
    ExecStop=/opt/nexus-3.15.2-01/bin/nexus stop
    User=nexus
    Restart=on-abort
      
    [Install]
    WantedBy=multi-user.target
    ```

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable nexus.service
    sudo systemctl start nexus.service
    ```

> 验证服务是否启动成功：`tail -f /opt/sonatype-work/nexus3/log/nexus.log`

> `./nexus-3.19.1-01/etc/nexus-default.properties`中定义了应用的`port`及`context-path`的默认值，默认访问URL为：http://127.0.0.1:8081  此文件不应该被修改，应修改`$data-dir/etc/nexus.properties`配置文件，此文件在Nexus首次启动时创建，你也可以在首次启动前手动创建。

> 默认admin的初始密码存储于`sonatype-work/nexus3/admin.password`文件中

#### UI 配置

1. 配置`Blob Stores` `Soft Quota`
    
    `管理界面` --> `Repository` --> `Blob Stores` --> `default`（默认Store）--> 选中`Enable Soft Quota` / `Type of Quota`: `Space Remaining` / `Quota Limit in MB`: `100` （可根据实际情况而定）
    
    有多少个Blob Store就配置多少次，一个Blob Store应挂载一个磁盘。
    
    超过配额会有警告信息，信息查看入口：
    * A WARN level message in the logs
    * In the Status item in the Support sub menu of the Administration menu, the status of all soft quotas is aggregated in the BlobStores status
    * A REST API endpoint
    
    > [参考文档](https://help.sonatype.com/repomanager3/configuration/repository-management#RepositoryManagement-AddingaSoftQuota)

2. Capabilities

    包含以下配置：
    * Audit - Enable audit of system changes - [链接](https://help.sonatype.com/repomanager3/configuration/auditing)
    * Base URL - Specify the reverse proxy base URL for your instance to be used in email notifications
    * NXRM2 style URLs
    * Storage Settings
    * UI
    * Upgrade
    * webhook
    * ...

    > [Base URL Creation](https://help.sonatype.com/repomanager3/configuration/system-configuration#SystemConfiguration-BaseURLCreation) - 配置BaseURL
    
    > [Capabilities参考文档](https://help.sonatype.com/repomanager3/configuration/system-configuration#SystemConfiguration-AccessingandConfiguringCapabilities)

3. Email/SMTP Server Configuration - [文档](https://help.sonatype.com/repomanager3/configuration/system-configuration#SystemConfiguration-Email/SMTPServerConfiguration)
    
4. [HTTP and HTTPS Request and Proxy Settings](https://help.sonatype.com/repomanager3/configuration/system-configuration#SystemConfiguration-HTTPandHTTPSRequestandProxySettings)

5. [Configuring and Executing Tasks](https://help.sonatype.com/repomanager3/configuration/system-configuration#SystemConfiguration-ConfiguringandExecutingTasks)
    
    常用定时任务：
    * Admin - Compact blob store - 物理删除已标记为删除状态的blob
    * Admin - Export databases for backup - 全量备份数据库数据：access logs、repository manager configuration、 security configuration。It is important to note this task does not backup actual repository content.
    * Admin - Execute script
    * Docker - Delete incomplete uploads
    * Maven - Delete SNAPSHOT
    * 其他"Repair -"开头的Task，建议创建为`Manual`(手动触发)
    
    > [各种Task的解释](https://help.sonatype.com/repomanager3/configuration/system-configuration#SystemConfiguration-TypesofTasksandWhentoUseThem)
    
    **Task Logging**
    
    The output of every task run will go to a separate log file. By default these task logs are stored in `$data-dir/log/tasks`. The file name of each task log is the type followed by the full date and time the task started. For example: `repository-maven.purge-unused-snapshots-20170618153235.log`.
    
    Generally the output of the task will go to both the `nexus.log` and the specific task log, however some tasks will only go to the `nexus.log`.
    
    For long running tasks, progress will be logged back to the `nexus.log` every 10 minutes as the work continues. Most commonly this will appear as a contextual update relevant to that particular task such as the number of items that have been processed, updated, deleted, etc.
    
    Task log files are removed after 30 days.

6. [Cleanup Policies -【重要】](https://help.sonatype.com/repomanager3/cleanup-policies)

    Anything deleted by cleanup policies is soft deleted. Disk space is not reclaimed until an Admin - Compact blob store task is run. Frequency and/or automation of that task should be done to your level of caution.
    
    **Cleanup Task**
    
    On start of a server which has cleanup abilities, a task named "Cleanup service" of type `Admin - Cleanup repositories using their associated policies` will be automatically created. By default, this task is scheduled to run `daily at 1AM` server time. Similar to other tasks, this task can be edited, disabled and executed manually if desired. If deleted, it will be automatically recreated on server restart. For more on tasks in general, see `Configuring and Executing Tasks`.

    When run, this task will execute cleanup of all the repositories which have a policy other than None set. There is no partial execution. This task cannot be manually created and either runs or does not.
    
    > [Docker Cleanup Strategies](https://help.sonatype.com/repomanager3/cleanup-policies#CleanupPolicies-DockerCleanupStrategies) - The `Docker cleanup policy` checks against the tagged components only. Only when the `Docker - Delete unused manifests and images task` has run will the cleanup be 'complete'.

    > [Investigating Blobstore and Repository Size and Space Usage](https://support.sonatype.com/hc/en-us/articles/115009519847?_ga=2.219523355.346890334.1575246796-534816866.1574671641) - 通过Groovy查询Repository的磁盘使用情况
    
    **Keeping Disk Usage Low** - [文档](https://help.sonatype.com/repomanager3/cleanup-policies/keeping-disk-usage-low)
    * Tasks - 通过定时任务删除
    * REST API - 通过调用REST API删除
    * Examining Blobstore Space Usage
    * Totally Out of Space / Seeing Errors?
        * [What to Do When the Database is Out of Disk Space](https://support.sonatype.com/hc/en-us/articles/360000052388?_ga=2.240034305.346890334.1575246796-534816866.1574671641)
        * [What to Do When the Blobstore is Out of Disk Space](https://support.sonatype.com/hc/en-us/articles/360000096228?_ga=2.240034305.346890334.1575246796-534816866.1574671641)
    
#### PID File

When started as a service on Linux, NXRM will create a file which holds a process ID in the operating system /tmp directory.  This file will have have a name of the form:

* prefix: "i4jdaemon_"
* suffix: the absolute path to the nexus start script with path part separators replaced with underscores
Example: Temporary directory is /tmp and path to start script is at /opt/nexus/nexus-3.14.0-04/bin/nexus , then the file created will be at "/tmp/i4jdaemon__opt_nexus_nexus-3.14.0-04_bin_nexus". 

**If the service pid file cannot be written the service startup will silently fail, without any logging statements written to the nexus.log.**

If the NXRM process is already stopped, and the service is failing to start, first confirm there is no log output being added to the nexus.log file. If not, then check for an pre-existing pid file. If that file is present, delete it first before trying to start the service.

The directory this file is created in can opttionally be changed by editing $install-dir/bin/nexus.vmoptions and adding a line like this:

```
-Dinstall4j.pidDir=/some/absolute/path/to/a/directory
```

The directory specified by the propery must already exist for the property to work - the directory will not be created automatically.


## Note

* [Blob Store 配置建议](https://help.sonatype.com/repomanager3/configuration/repository-management#RepositoryManagement-ChoosingtheNumberofBlobStores)

    The most flexible approach is to create a separate blob store for each repository, although this is not recommended except in extreme cases of unpredictable capacity because of the administrative complexity. 
    
    The current repository-to-blob store limitations will be removed in an upcoming release of Nexus Repository Manager, which will make it possible to revise your repository/blob store approach over time.

## 文档

* [官方文档](https://help.sonatype.com/docs)
    * [Quick Start Guide - Proxying Maven and NPM](https://help.sonatype.com/repomanager3/quick-start-guide---proxying-maven-and-npm)
    * [Run Behind a Reverse Proxy](https://help.sonatype.com/repomanager3/installation/run-behind-a-reverse-proxy) - Apache、Nginx配置
    * [Nexus Directories简介](https://help.sonatype.com/repomanager3/installation/directories)
    * [Searching for Components](https://help.sonatype.com/repomanager3/user-interface/searching-for-components) - 搜索功能
    
    * [Repository Management](https://help.sonatype.com/repomanager3/configuration/repository-management)
        * Blob Stores
            * Choosing the Number of Blob Stores
            * Estimating Blob Store Size
            * Choosing the Blob Store Type
            * Adding a Soft Quota
        * Managing Repositories and Repository Groups
        * Content Selectors
    
    * [Storage Guide](https://help.sonatype.com/repomanager3/configuration/storage-guide) - 讲解Blob相关的术语及`Blob Store Types`
        * [Relocating Blob Stores - 【重要】](https://support.sonatype.com/hc/en-us/articles/235816228-Relocating-Blob-Stores?_ga=2.210608791.346890334.1575246796-534816866.1574671641) - 当你不是high availability cluster部署时，请使用此方案`迁移Blob Stores`数据，非收费版本使用此方案。
    
    * [Backup and Restore](https://help.sonatype.com/repomanager3/backup-and-restore)
        * [Prepare a Backup](https://help.sonatype.com/repomanager3/backup-and-restore/prepare-a-backup)
        * [Configure and Run the Backup Task](https://help.sonatype.com/repomanager3/backup-and-restore/configure-and-run-the-backup-task)
        * [Restore Exported Databases](https://help.sonatype.com/repomanager3/backup-and-restore/restore-exported-databases)
        * [Designing your Cluster Backup/Restore Process](https://help.sonatype.com/repomanager3/high-availability/designing-your-cluster-backup-restore-process) - 仅做参考，因为这是集群恢复方案
    
    * [Routing Rules](https://help.sonatype.com/repomanager3/configuration/routing-rules)

    * [Security](https://help.sonatype.com/repomanager3/security)
        * [Realms](https://help.sonatype.com/repomanager3/security/realms)
        * [Privileges](https://help.sonatype.com/repomanager3/security/privileges)
        * [LDAP](https://help.sonatype.com/repomanager3/security/ldap)
        * [Authentication via Remote User Token](https://help.sonatype.com/repomanager3/security/authentication-via-remote-user-token)
        * [Configuring SSL](https://help.sonatype.com/repomanager3/security/configuring-ssl)
            * Outbound SSL - Trusting SSL Certificates of Remote Repositories
            * Outbound SSL - Trusting SSL Certificates Globally
            * Outbound SSL - Trusting SSL Certificates Using Keytool
            * Inbound SSL - Configuring to Serve Content via HTTPS
                * Using A Reverse Proxy Server
                * Serving SSL Directly
    
    * [Formats -【重要】](https://help.sonatype.com/repomanager3/formats) - Nexus支持的仓库类型
        * Docker Registry
            * [Using Self-Signed Certificates with Nexus Repository Manager and Docker Daemon](https://support.sonatype.com/hc/en-us/articles/217542177?_ga=2.256297225.346890334.1575246796-534816866.1574671641)
            * [Docker Repository Reverse Proxy Strategies](https://support.sonatype.com/hc/en-us/articles/360000761828-Docker-Repository-Reverse-Proxy-Strategies?_ga=2.206481681.346890334.1575246796-534816866.1574671641)
            
    * [REST and Integration API](https://help.sonatype.com/repomanager3/rest-and-integration-api)
    
    * [Webhooks](https://help.sonatype.com/repomanager3/webhooks)
    
    * [Bundle Development](https://help.sonatype.com/repomanager3/bundle-development) - Nexus Repository Manager is built on top of the OSGi container Apache Karaf. The functionality is encapsulated in a number of OSGi bundles.
    
    * [High Availability](https://help.sonatype.com/repomanager3/high-availability) - 仅商业版支持
    
    * [Upgrading](https://help.sonatype.com/repomanager3/upgrading)
        * [How do I upgrade Nexus Repository Manager 2.x to 2.y](https://support.sonatype.com/hc/en-us/articles/213464138?_ga=2.18802363.346890334.1575246796-534816866.1574671641)
        * [Upgrading Nexus Repository Manager 3.1.0 and Newer](https://support.sonatype.com/hc/en-us/articles/115000350007?_ga=2.240635777.346890334.1575246796-534816866.1574671641)
         * [Upgrade Procedures - 从2.x升级至3.x](https://help.sonatype.com/repomanager3/upgrading/upgrade-procedures)
        


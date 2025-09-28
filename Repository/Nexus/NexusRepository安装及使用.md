# NexusRepository安装及使用

## 安装

> [Nexus下载地址](https://help.sonatype.com/en/download.html)

### 1. 系统要求

`mount`建议使用`noatime`配置，以提供性能。
* [Optimizing Nexus Disk IO Performance](https://support.sonatype.com/hc/en-us/articles/213465258-Optimizing-Nexus-Disk-IO-Performance?_ga=2.138188403.2134597265.1574671641-534816866.1574671641) - Avoid Recording File Access Times On Reads

* [系统需求总览](https://help.sonatype.com/en/sonatype-nexus-repository-system-requirements.html) : CPU、内存、Java版本、数据库、磁盘、文件系统。
  * [Required Trigram Module for PostgreSQL Database](https://help.sonatype.com/en/sonatype-nexus-repository-system-requirements.html#database-requirements)
  * [Installing the Trigram Module](https://help.sonatype.com/en/installing-the-trigram-module.html)
  * [Required PostgreSQL Database User Permissions](https://help.sonatype.com/en/sonatype-nexus-repository-system-requirements.html#database-requirements)
  * [Adjust the PostgreSQL Max Connections](https://help.sonatype.com/en/postgresql-max-connections.html)
  * [Advanced Database Memory Tuning](https://help.sonatype.com/en/advanced-database-memory-tuning.html)
  * [Nexus Repository Memory Overview](https://help.sonatype.com/en/nexus-repository-memory-overview.html)【重要，根据实际情况调整JVM参数】

#### 1.1 File Handle Limits

**init.d**

当使用`init.d`管理文件句柄限制时。

a. 在`/etc/security/limits.conf`文件中添加以下配置，`userid`为运行Nexus的用户。
```
<userid> - nofile 65536
<userid> - nproc 65536
```

b. (可选)当使用`Ubuntu`时，进程是通过`init.d`运行的，则会忽略`limits.conf`文件。编辑`/etc/pam.d/common-session`文件，去掉以下注释即可。
```
# session    required   pam_limits.so
```

c. 重启服务。

**systemd**

当使用`systemd`运行服务时，修改配置文件增加`LimitNOFILE`配置，然后重启服务：
```shell
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNPROC=65536
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

### 2. Install And Run

#### 2.1安装及配置

```bash
# 安装到 /opt/sonatype 目录
shell> mkdir /opt/sonatype && cd /opt/sonatype

# 解压出 "nexus" --> 应用程序文件夹 。"sonatype-work" --> 数据文件夹
shell> tar xvz --keep-directory-symlink -f nexus-3.84.0-03-linux-x86_64.tar.gz
# 链接 $installdir 到 /op/sonatypet/nexus
shell> ln -s /opt/sonatype/nexus-3.84.0-03 /opt/sonatype/nexus

# 添加nexus用户及用户组
shell> useradd nexus

# 修改 run_as_user="nexus" ，并去除前面的注释
shell> vim ./nexus/bin/nexus.rc

shell> chown -R nexus:nexus /opt/sonatype

# 后台运行Nexus，日志输出到日志文件。后台运行指令：
#   - start
#   - stop
#   - restart
#   - force-reload
#   - status
shell> ./nexus/bin/nexus start

# 在当前shell启动Nexus：日志会输出到控制台，按"CTRL-C"会关闭Nexus
shell> ./nexus/bin/nexus run
```

> - [JVM及应用配置](https://help.sonatype.com/en/configuring-the-runtime-environment.html) -【重要】
> - [重试机制的配置](https://help.sonatype.com/repomanager3/configuration/retry-limit-configuration) - 可选配置

#### 2.2 Run as a Service

创建`/etc/systemd/system/nexus.service`，文件内容：

```
[Unit]
Description=nexus service
After=network.target
  
[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/sonatype/nexus/bin/nexus start
ExecStop=/opt/sonatype/nexus/bin/nexus stop

User=nexus
Restart=on-abort
TimeoutSec=600
  
[Install]
WantedBy=multi-user.target
```

```bash
# 激活服务
sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service
```

> * 修改JVM参数 `./nexus/bin/nexus.vmoptions`
> * 验证服务是否启动成功 `tail -f /opt/sonatype-work/nexus3/log/nexus.log`
> * `./nexus/etc/nexus-default.properties`中定义了应用的`port`及`context-path`的默认值，默认访问URL为：http://127.0.0.1:8081 。
> 此文件不应该被修改，应修改`$data-dir/etc/nexus.properties`配置文件，此文件在Nexus首次启动时创建，你也可以在首次启动前手动创建。
> * `$data-dir` 默认值为`../sonatype-work/nexus3`
> * 默认admin的初始密码存储于`sonatype-work/nexus3/admin.password`文件中


#### 2.3 配置 PostgreSQL

- [Install Nexus Repository with a PostgreSQL Database](https://help.sonatype.com/en/install-nexus-repository-with-postgresql.html)

#### 2.4 Nginx配置

- [Run Behind a Reverse Proxy](https://help.sonatype.com/en/run-behind-a-reverse-proxy.html)


#### 2.5 UI 配置

1. 配置`Blob Stores` `Soft Quota`
    
    `管理界面` --> `Repository` --> `Blob Stores` --> `default`（默认Store）--> 选中`Enable Soft Quota` / `Type of Quota`: `Space Remaining` / `Quota Limit in MB`: `100` （可根据实际情况而定）
    
    有多少个Blob Store就配置多少次，一个Blob Store应挂载一个磁盘。
    
2. [Tasks](https://help.sonatype.com/en/tasks.html)
    
    常用定时任务：
    * Admin - Compact blob store - 物理删除已标记为删除状态的blob
    * Admin - Export databases for backup - 全量备份数据库数据：access logs、repository manager configuration、 security configuration。It is important to note this task does not backup actual repository content.
    * Admin - Execute script
    * Docker - Delete incomplete uploads
    * Maven - Delete SNAPSHOT
    * 其他"Repair -"开头的Task，建议创建为`Manual`(手动触发)
    
3. [Cleanup Policies](https://help.sonatype.com/en/cleanup-policies.html)

    Anything deleted by cleanup policies is soft deleted. Disk space is not reclaimed until an Admin - Compact blob store task is run. Frequency and/or automation of that task should be done to your level of caution.
    
    > [Investigating Blobstore and Repository Size and Space Usage](https://support.sonatype.com/hc/en-us/articles/115009519847?_ga=2.219523355.346890334.1575246796-534816866.1574671641) - 通过Groovy查询Repository的磁盘使用情况
    
    **Keeping Disk Usage Low** - [文档](https://help.sonatype.com/en/keeping-disk-usage-low.html)
    * Tasks - 通过定时任务删除
    * REST API - 通过调用REST API删除
    * Examining Blobstore Space Usage
    * Totally Out of Space / Seeing Errors?
        * [What to Do When the Database is Out of Disk Space](https://support.sonatype.com/hc/en-us/articles/360000052388?_ga=2.240034305.346890334.1575246796-534816866.1574671641)
        * [What to Do When the Blobstore is Out of Disk Space](https://support.sonatype.com/hc/en-us/articles/360000096228?_ga=2.240034305.346890334.1575246796-534816866.1574671641)
    

## 文档

* [Sonatype Support Knowledge Base](https://support.sonatype.com/hc/en-us) - 搜索Sonatype的知识分享，协助解决问题。
* [官方文档](https://help.sonatype.com/en/sonatype-nexus-repository.html)
  * [Directories](https://help.sonatype.com/en/directories.html) - 目录结构说明
  * [重置管理员密码](https://help.sonatype.com/en/reset-the-admin-password.html)
  * [UI`搜索`功能详解](https://help.sonatype.com/en/searching-for-components.html)
  * [UI上传组件](https://help.sonatype.com/en/uploading-components.html)
  * [npm身份认证](https://help.sonatype.com/en/npm-security.html)
  * [Nexus API接口](https://help.sonatype.com/en/automation.html)




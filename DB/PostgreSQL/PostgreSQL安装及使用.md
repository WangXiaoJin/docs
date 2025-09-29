# PostgreSQL安装及使用

## ㈠ 安装

### 1. 安装

下面是在`AlmaLinux 8`的`x86_64`架构上安装`PostgreSQL 17`的步骤，详情请[参考官网](https://www.postgresql.org/download/linux/redhat/)。
请用`root用户`执行以下命令：

```shell
# Install the repository RPM:
$ dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Disable the built-in PostgreSQL module:
$ dnf -qy module disable postgresql

# 自动创建用户及组：`postgres:x:26:26:PostgreSQL Server:/var/lib/pgsql:/bin/bash`和 `postgres:x:26:`
# `/var/lib/pgsql` 的 home目录中存在 `.bash_profile` 文件，里面默认设置了环境变量 `PGDATA=/var/lib/pgsql/17/data`
$ dnf install -y postgresql17-server

# 创建归档数据文件目录
$ mkdir -p /apply/pgsql/17/archive_wal
# 设置目录权限
$ chown -R postgres:postgres /apply/pgsql 
$ chmod -R go-rwx /apply/pgsql

# 查看 postgres 组的ID
$ cat /etc/group | grep postgres
# 设置 vm.nr_hugepages，假如 postgres 组的ID为 26
$ echo "vm.hugetlb_shm_group=26" | sudo tee -a /etc/sysctl.conf
$ sysctl -p

# 修改服务中的配置。设置`PGDATA`、`maxproc`、`openfiles`和`datasize`
# LimitMEMLOCK 根据 shared_buffers 配置进行调整，可适当大于 shared_buffers
$ vim /usr/lib/systemd/system/postgresql-17.service
[Service]
LimitNOFILE=1048576
LimitNPROC=infinity
LimitDATA=infinity
LimitMEMLOCK=8G
Environment=PGDATA=/apply/pgsql/17/data/
$ systemctl daemon-reload

# locale -a 显示所有可配置的locale。lc-collate和lc-ctyp值为非C或POSIX时，会影响性能。
# 建议 locale 设置为 en_US.utf8，修改后需新建窗口才生效
# 查看 locale 配置
$ locale
# 设置 locale 为 en_US.utf8
$ localectl set-locale LANG=en_US.utf8

# `postgresql-17-setup initdb` 实际执行的命令为：`initdb --pgdata='$PGDATA' -A scram-sha-256 --auth-local=peer $PGSETUP_INITDB_OPTIONS`
# 所以我们可以借助 $PGSETUP_INITDB_OPTIONS 环境变量来设置初始化数据库的选项。
$ export PGSETUP_INITDB_OPTIONS="--encoding=UTF8 --data-checksums --locale=C --lc-monetary='zh_CN.UTF-8' --lc-numeric='zh_CN.UTF-8' --lc-time='zh_CN.UTF-8'"
$ /usr/pgsql-17/bin/postgresql-17-setup initdb

$ systemctl enable postgresql-17
$ systemctl start postgresql-17
```

验证配置：
```shell
$ ps aux | grep postgres
$ cat /proc/<PID>/limits | grep -i "open files\|processes\|data"
```

> - [内核资源配置](https://www.postgresql.org/docs/current/kernel-resources.html)
>   - Shared Memory and Semaphores
>   - systemd RemoveIPC - 如果是自己创建postgres用户时，需要设置为系统用户: `adduser --system postgres`
>   - Resource Limits
>   - Linux Memory Overcommit
>   - Linux Huge Pages

### 2. 数据库配置

`postgresql.conf`配置（Primary）：
```shell
# 数据库集群名称
cluster_name = 'pg_primary'
# 集群配置，非集群部署请注释掉此配置
#synchronous_standby_names = 'ANY 1 (pg_standby)'
archive_mode = on
archive_command = 'test ! -f /apply/pgsql/17/archive_wal/%f && cp %p /apply/pgsql/17/archive_wal/%f'

listen_addresses = '0.0.0.0'
port = 5432
max_connections = 2000

tcp_keepalives_idle = 10min
tcp_keepalives_interval = 10s
tcp_keepalives_count = 3
tcp_user_timeout = 30s

# 缓存普通表和索引数据。物理内存的 25%，但通常不超过 32GB
shared_buffers = 8GB
# 仅用于当前会话创建的临时表
temp_buffers = 32MB
# 每个操作私有，用于查询操作的排序、哈希等
work_mem = 16MB

# 期望可以同时执行的并发磁盘I/O操作的数量。如果你使用的是SSD或NVMe存储设备，建议设置为100 - 200。
#effective_io_concurrency = 1
max_worker_processes = 8
max_parallel_workers = 8
max_parallel_maintenance_workers = 2
max_parallel_workers_per_gather = 2

log_destination = 'stderr'
logging_collector = on
# 配置文件名 %d_%H。每小时1个文件，保留一个月
log_filename = 'postgresql-%d_%H.log'
log_rotation_age = 1h
# 禁用文件大小旋转，避免影响文件名生成或日志覆盖
log_rotation_size = 0
log_truncate_on_rotation = on
# 默认值: WARNING
#log_min_messages = INFO
# 默认值：ERROR
#log_min_error_statement = WARNING
log_min_duration_statement = 10s
log_connections = on
log_disconnections = on
log_error_verbosity = verbose
log_line_prefix = '%m [%b %p:%c] %q[%a/%u@%h/%d] '
#log_lock_waits = on
#log_recovery_conflict_waits = on
# 此设置的非零值会增加开销，因为无论最终是否发生错误， PostgreSQL都需要在每个语句开始时将参数值的文本表示存储在内存中。
#log_parameter_max_length_on_error = -1
log_replication_commands = on
#log_temp_files = -1
# 中华人民共和国（People's Republic of China）
log_timezone = 'Asia/Shanghai'

statement_timeout = 20s
# 获取锁超时时间，必须小于 statement_timeout
lock_timeout = 18s
idle_in_transaction_session_timeout = 5min
datestyle = 'iso, ymd'
timezone = 'Asia/Shanghai'

client_encoding = 'UTF8'
lc_monetary = 'zh_CN.UTF-8'
lc_numeric = 'zh_CN.UTF-8'
lc_time = 'zh_CN.UTF-8'
```

修改`pg_hba.conf`配置，允许远程连接数据库服务：
```shell
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             0.0.0.0/0             scram-sha-256
# IPv6 local connections:
host    all             all             ::0/0                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             0.0.0.0/0             scram-sha-256
host    replication     all             ::0/0                 scram-sha-256
```

配置修改好后重启服务，并执行以下操作：
```shell
# 修改 postgres 密码
$ sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'postgres@765';"

# 验证内核是否配置 CONFIG_HUGETLBFS=y 、CONFIG_HUGETLB_PAGE=y
$ grep -E "CONFIG_HUGETLBFS|CONFIG_HUGETLB_PAGE" /boot/config-$(uname -r)
# 查看当前 shared_buffers 的配置值需要多少 hugepages
$ sudo -u postgres psql -c "SHOW shared_memory_size_in_huge_pages;"
 shared_memory_size_in_huge_pages
----------------------------------
 4262
(1 row)

# 查看当前系统的 hugepagesize 配置为 2048 kB
$ grep ^Hugepagesize /proc/meminfo
Hugepagesize:       2048 kB

# 当前 Session 生效
$ sysctl -w vm.nr_hugepages=4262
# 修改 hugepages-2048kB 配置为 4262，重启服务后生效
$ echo 4262 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

# 重启服务
$ systemctl restart postgresql-17
```

**以下为集群配置**

创建集群复制账号及赋权：
```shell
# 执行SQL
CREATE USER replicator WITH REPLICATION PASSWORD 'your_secure_password';
```

Standby增加以下配置：
```shell
cluster_name = 'pg_standby'
restore_command = 'cp /apply/pgsql/17/archive_wal/%f %p'
primary_conninfo = 'host=primary_host port=5432 user=replicator password=secret'
# 需进一步验证使用的必要性（有些三方工具会自动配置）。需要主库创建物理复制槽点：pg_standby_slot（SELECT * FROM pg_create_physical_replication_slot('pg_standby_slot');）
#primary_slot_name = 'pg_standby_slot'
# 默认 on
#hot_standby = on
hot_standby_feedback = on
# 高可用性的纯备份服务器建议调低，作为异步同步数据的只读库则可以调大
max_standby_archive_delay = 5s
max_standby_streaming_delay = 5s

# 启用逻辑复制槽功能时可配置此参数。当主库挂了，从库提升为主库后，逻辑复制库可自然切换至新的主库同步数据。配合 synchronized_standby_slots 一起使用。
#sync_replication_slots = on
```

> 注：配置项中的WAL归档目录为：`/apply/pgsql/17/archive_wal/`，推荐使用NFS或S3存储，以便主备时时共享文件。

`参数配置`相关文档：
- [连接和认证](https://www.postgresql.org/docs/current/runtime-config-connection.html)
- [资源](https://www.postgresql.org/docs/current/runtime-config-resource.html)
- [WAL - Write Ahead Log](https://www.postgresql.org/docs/current/runtime-config-wal.html)
- [`streaming replication` AND `logical replication`](https://www.postgresql.org/docs/current/runtime-config-replication.html)
- [日志](https://www.postgresql.org/docs/current/runtime-config-logging.html)
- [统计](https://www.postgresql.org/docs/current/runtime-config-statistics.html)
- [Client Connection Defaults](https://www.postgresql.org/docs/current/runtime-config-client.html)
- [Lock Management](https://www.postgresql.org/docs/current/runtime-config-locks.html)


### 3. 创建role及授权

PostgreSQL使用`role`的概念来管理数据库访问权限。根据角色的设置方式，可以将角色视为一个`user`或`group`。

```shell
# 切换到 postgres 用户
$ su - postgres

$ createuser name
$ CREATE ROLE jonathan LOGIN;
$ CREATE USER davide WITH PASSWORD 'jw8s0F4';
```

- 通过 [pg_hba.conf](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html) 配置文件授权。
- [Password Authentication](https://www.postgresql.org/docs/current/auth-password.html)
- [Roles 管理](https://www.postgresql.org/docs/current/user-manag.html)
- [预定义角色列表](https://www.postgresql.org/docs/current/predefined-roles.html)

### 4. 刷新配置
```shell
# 方法1：使用 pg_ctl（需要知道数据目录）
$ sudo -u postgres pg_ctl reload -D ${PGDATA}

# 方法2：使用 psql 发送信号
$ sudo -u postgres psql -c "SELECT pg_reload_conf();"

# 方法3：使用 systemctl
$ systemctl reload postgresql
```

## ㈡ 维护

### 1. 备份和恢复、高可用、负载均衡、复制

- [High Availability, Load Balancing, and Replication](https://www.postgresql.org/docs/current/high-availability.html)
- [Backup and Restore](https://www.postgresql.org/docs/current/backup.html)
- [Recovery参数配置](https://www.postgresql.org/docs/current/runtime-config-wal.html#RUNTIME-CONFIG-WAL-ARCHIVE-RECOVERY)
- [Replication 配置](https://www.postgresql.org/docs/current/runtime-config-replication.html)

> 从基础备份恢复数据库时需要删除以下文件：
> 1. postmaster.pid、postmaster.opts 文件。
> 2. pg_dynshmem/, pg_notify/, pg_serial/, pg_snapshots/, pg_stat_tmp/ 和 pg_subtrans/ 目录下的文件，服务启动时会创建这些文件。
> 3. pg_replslot/ 目录下的文件，以防影响WAL文件的清理。
> 4. 以 pgsql_tmp 开头的文件或目录，以及pg_internal.init文件。

> At startup, the standby begins by restoring all WAL available in the archive location, calling `restore_command`. 
> Once it reaches the end of WAL available there and `restore_command` fails, it tries to restore any WAL available in the `pg_wal` directory. 
> If that fails, and streaming replication has been configured, the standby tries to connect to the primary server and 
> start streaming WAL from the last valid record found in archive or pg_wal. If that fails or streaming replication is not configured, 
> or if the connection is later disconnected, the standby goes back to step 1 and tries to restore the file from the archive again. 
> This loop of retries from the archive, `pg_wal`, and via streaming replication goes on until the server is stopped or is promoted.
> 
> Standby mode is exited and the server switches to normal operation when `pg_ctl promote` is run, or `pg_promote()` is called. 
> Before failover, any WAL immediately available in the archive or in pg_wal will be restored, but no attempt is made to connect to the primary.
> 
> If `archive_mode` is set to `on`, the archiver is not enabled during recovery or standby mode. If the standby server is promoted, 
> it will start archiving after the promotion, but will not archive any WAL or timeline history files that it did not generate itself. 
> To get a complete series of WAL files in the archive, you must ensure that all WAL is archived, before it reaches the standby. 
> This is inherently true with file-based log shipping, as the standby can only restore files that are found in the archive, 
> but not if streaming replication is enabled. When a server is not in recovery mode, there is no difference between `on` and `always` modes.

### 2. 分布式、高可用(三方方案)

- Patroni
- Repmgr
- Citus
- PgBouncer
- Pgpool-II

### 3. 监控数据库

- [Monitoring Database Activity](https://www.postgresql.org/docs/current/monitoring.html)


## ㈢文档

- [官方文档](https://www.postgresql.org/docs/current/index.html)
- [SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html) - 包含所有指令


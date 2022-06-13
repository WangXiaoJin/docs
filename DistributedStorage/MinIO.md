# MinIO

## 安装及配置

### 安装实践

假设有三台机器`192.168.201.11`、`192.168.201.12`、`192.168.201.13`，每台机器挂载4块磁盘，这四块磁盘分别挂载`/data01`、`/data02`、`/data03`、`/data04`目录。

操作步骤如下：

* 所有服务器执行性能调优 - [Kernel Tuning for MinIO Production Deployment on Linux Servers](https://github.com/minio/minio/tree/master/docs/deployment/kernel-tuning)
此脚本是临时修改配置，重启后配置会丢失，你可以直接修改文件永久生效

* 三台机器`/etc/hosts`都增加以下配置
    ```
    192.168.201.11 minio01
    192.168.201.12 minio02
    192.168.201.13 minio03
    ```

* 创建`/usr/minio`目录，和minio相关的命令放在此目录中。配置环境变量`/etc/profile`
    ```shell script
    MINIO_ACCESS_KEY=admin
    MINIO_SECRET_KEY=admin123
    export MINIO_ACCESS_KEY MINIO_SECRET_KEY
    
    export PATH=/usr/minio:$PATH
    ```

* 三台机器安装Server，执行以下相同命令
    ```shell script
    wget https://dl.min.io/server/minio/release/linux-amd64/minio
    chmod +x minio
    # 端口不配时默认使用 9000
    nohup minio server http://minio0{1...3}/data0{1...4} &
    ```
    启动成功后就可以通`http://192.168.201.11:9000`、`http://192.168.201.12:9000`、`http://192.168.201.13:9000`访问。

* 在任意一台机器安装 MinIO Client
    ```shell script
    wget https://dl.min.io/client/mc/release/linux-amd64/mc
    chmod +x mc
    # 启用Shell自动补全功能
    mc --autocompletion
    # 配置alias到刚才新建的集群，名字随便定义，下面我们使用的alias名字为`self`
    mc alias set self http://192.168.201.11:9000 admin admin123
    # 创建bucket，名字：mybucket 
    mc mb self/mybucket
    # 拷贝本地文件至MinIO集群的mybucket
    mc cp img-01.jpg self/mybucket
    # 磁盘损坏时需要修复数据，用下面的命令（注：命令已废弃，MinIO会后台自动修复）
    mc admin heal -r self/mybucket
    ```


### 安装及配置参考文档

* 安装
    * [安装及升级](https://docs.min.io/docs/minio-quickstart-guide.html)
    * [Docker启动MinIO](https://docs.min.io/docs/minio-docker-quickstart-guide.html)
    * [MinIO Deployment Quickstart Guide](https://docs.min.io/docs/minio-deployment-quickstart-guide.html)
        * [Deploy MinIO on Docker Swarm](https://docs.min.io/docs/deploy-minio-on-docker-swarm.html)
        * [Deploy MinIO on Kubernetes](https://docs.min.io/docs/deploy-minio-on-kubernetes.html)
        * [Deploy MinIO on Docker Compose](https://docs.min.io/docs/deploy-minio-on-docker-compose.html)
    * [Kernel Tuning for MinIO Production Deployment on Linux Servers -【重要】](https://github.com/minio/minio/tree/master/docs/deployment/kernel-tuning) - Linux系统调优
    * 扩展
        * [Federation Quickstart Guide](https://github.com/minio/minio/tree/master/docs/federation/lookup)
        * [Deploy MinIO on Chrooted Environment](https://github.com/minio/minio/tree/master/docs/chroot)

* Distributed
    * [Distributed Server Design Guide -【重要】](https://github.com/minio/minio/blob/master/docs/distributed/DESIGN.md)
    * [Distributed MinIO Quickstart Guide -【重要】](https://docs.minio.io/docs/distributed-minio-quickstart-guide.html)
        * Data protection
        * High availability
        * Consistency Guarantees
        * 安装注意事项
    * [MinIO 的分布式部署](https://segmentfault.com/a/1190000022524583) - 仅作参考

* [MinIO Server Limits Per Tenant -【重要】](https://docs.min.io/docs/minio-server-limits-per-tenant.html)
* [MinIO Server Config Guide -【重要】](https://docs.min.io/docs/minio-server-configuration-guide.html)
    * [MinIO Admin Complete Guide](https://github.com/minio/mc/blob/master/docs/minio-admin-complete-guide.md)
    * [Bucket Quota Configuration Quickstart Guide](https://github.com/minio/minio/tree/master/docs/bucket/quota)
    * [Compression Guide](https://github.com/minio/minio/tree/master/docs/compression)
    * [MinIO Server Debugging Guide](https://github.com/minio/minio/tree/master/docs/debugging)
    * [MinIO Logging Quickstart Guide](https://github.com/minio/minio/tree/master/docs/logging)
    * [MinIO Server Throttling Guide](https://github.com/minio/minio/tree/master/docs/throttle)

* [MinIO Erasure Code Quickstart Guide](https://docs.min.io/docs/minio-erasure-code-quickstart-guide.html)
    * [MinIO Storage Class Quickstart Guide](https://github.com/minio/minio/tree/master/docs/erasure/storage-class)

* [MinIO Monitoring Guide](https://docs.min.io/docs/minio-monitoring-guide.html)
    * [MinIO Healthcheck](https://github.com/minio/minio/blob/master/docs/metrics/healthcheck/README.md)
    * [How to monitor MinIO server with Prometheus](https://docs.min.io/docs/how-to-monitor-minio-using-prometheus.html)
    * [How to monitor MinIO server with Grafana](https://github.com/minio/minio/blob/master/docs/metrics/prometheus/grafana/README.md)

* Secure
    * [How to secure access to MinIO server with TLS](https://docs.min.io/docs/how-to-secure-access-to-minio-server-with-tls.html)
        * [How to secure access to MinIO on Kubernetes with TLS](https://github.com/minio/minio/tree/master/docs/tls/kubernetes)
    * [MinIO Security Overview](https://docs.min.io/docs/minio-security-overview.html)
    * [MinIO Multi-user Quickstart Guide](https://docs.min.io/docs/minio-multi-user-quickstart-guide.html) - 创建User、Group、Policy
        * [MinIO Admin Multi-user Quickstart Guide](https://github.com/minio/minio/tree/master/docs/multi-user/admin)
    * [MinIO STS (Security Token Service) Quickstart Guide](https://github.com/minio/minio/tree/master/docs/sts) -
    Enables clients to request temporary credentials for MinIO resources
        * [AssumeRole](https://github.com/minio/minio/blob/master/docs/sts/assume-role.md)
        * [AssumeRoleWithClientGrants](https://github.com/minio/minio/blob/master/docs/sts/client-grants.md)
        * [Dex Quickstart Guide](https://github.com/minio/minio/blob/master/docs/sts/dex.md)
        * [etcd V3 Quickstart Guide](https://github.com/minio/minio/blob/master/docs/sts/etcd.md)
        * [Keycloak Quickstart Guide](https://github.com/minio/minio/blob/master/docs/sts/keycloak.md)
        * [AssumeRoleWithLDAPIdentity](https://github.com/minio/minio/blob/master/docs/sts/ldap.md)
        * [AssumeRoleWithWebIdentity](https://github.com/minio/minio/blob/master/docs/sts/web-identity.md)
        * [WSO2 Quickstart Guide](https://github.com/minio/minio/blob/master/docs/sts/wso2.md)
    * [KMS Guide](https://docs.min.io/docs/minio-kms-quickstart-guide.html)
    * [KES](https://github.com/minio/kes)  
        KES is a stateless and distributed key-management system for high-performance applications. We built KES as the
        bridge between modern applications - running as containers on Kubernetes - and centralized KMS solutions.
    * [How to use MinIO's Server-side Encryption with the AWS CLI](https://docs.min.io/docs/how-to-use-minio-s-server-side-encryption-with-aws-cli.html)
    * [Generate Let's Encrypt certificate using Certbot for MinIO](https://docs.min.io/docs/generate-let-s-encypt-certificate-using-concert-for-minio.html)

* MinIO Client
    * [MinIO Client Quickstart Guide -【重要】](https://docs.min.io/docs/minio-client-quickstart-guide.html)
    * [MinIO Client Complete Guide -【重要】](https://docs.min.io/docs/minio-client-complete-guide.html)
    * [MinIO Admin Complete Guide -【重要】](https://docs.min.io/docs/minio-admin-complete-guide.html)

* SDK
    * Java
        * [Java Client Quickstart Guide](https://docs.min.io/docs/java-client-quickstart-guide.html)
        * [Java Client API Reference](https://docs.min.io/docs/java-client-api-reference.html)
        * [Examples](https://github.com/minio/minio-java/tree/release/examples)
        * [Build your own Photo API Service - Full Application Example](https://github.com/minio/minio-java-rest-example)


## 注意事项

### HighwayHash

MinIO's erasure coded backend uses high speed [HighwayHash](https://github.com/minio/highwayhash) checksums to protect against Bit Rot.

[HighwayHash](https://github.com/minio/highwayhash) is a pseudo-random-function (PRF) developed by Jyrki Alakuijala, Bill Cox and Jan Wassenberg (Google research).
HighwayHash takes a 256 bit key and computes 64, 128 or 256 bit hash values of given messages.

It can be used to prevent hash-flooding attacks or authenticate short-lived messages. Additionally it can be used as a
fingerprinting function. HighwayHash is not a general purpose cryptographic hash function (such as Blake2b, SHA-3 or SHA-2)
and should not be used if strong collision resistance is required.

### How are drives used for Erasure Code?

MinIO divides the drives you provide into erasure-coding sets of 4 to 16 drives. Therefore, the number of drives you present
must be a multiple of one of these numbers. Each object is written to a single erasure-coding set.

Minio uses the largest possible EC set size which divides into the number of drives given. For example, 18 drives are
configured as 2 sets of 9 drives, and 24 drives are configured as 2 sets of 12 drives. This is true for scenarios when
running MinIO as a standalone erasure coded deployment. In [distributed setup however node (affinity) based](https://docs.minio.io/docs/distributed-minio-quickstart-guide.html)
erasure stripe sizes are chosen.

The drives should all be of approximately the same size.

### Bucket Versioning Guide

MinIO versioning is designed to keep multiple versions of an object in one bucket. For example, you could store
`spark.csv` (version `ede336f2`) and `spark.csv` (version `fae684da`) in a single bucket. Versioning protects you from unintended
overwrites, deletions, to apply retention policies and archive your objects.

> [Bucket Versioning Guide](https://docs.min.io/docs/minio-bucket-versioning-guide.html)

> [Bucket Versioning Design Guide](https://github.com/minio/minio/blob/master/docs/bucket/versioning/DESIGN.md)

### Object Lock and Immutablity Guide

MinIO server allows WORM for specific objects or by configuring a bucket with default object lock configuration that
applies default retention mode and retention duration to all objects. This makes objects in the bucket immutable
i.e. delete of the version are not allowed until an expiry specified in the bucket's object lock configuration or object retention.

Object locking requires locking to be enabled on a bucket at the time of bucket creation, object locking also
automatically enables versioning on the bucket. In addition, a default retention period and retention mode can be
configured on a bucket to be applied to objects created in that bucket.

Independent of retention, an object can also be under legal hold. This effectively disallows all deletes of an object
under legal hold until the legal hold is removed by an API call.

> [Object Lock and Immutablity Guide](https://docs.min.io/docs/minio-bucket-object-lock-guide.html)

### Bucket Replication Guide

Bucket replication is designed to replicate selected objects in a bucket to a destination bucket.

To replicate objects in a bucket to a destination bucket on a target site either in the same cluster or a different
cluster, start by enabling versioning for both source and destination buckets. Finally, the target site and the destination
bucket need to be configured on the source MinIO server.

> [Bucket Replication Guide](https://docs.min.io/docs/minio-bucket-replication-guide.html)

### MinIO Bucket Notification Guide

Events occurring on objects in a bucket can be monitored using bucket event notifications.

> [MinIO Bucket Notification Guide](https://docs.min.io/docs/minio-bucket-notification-guide.html)

### Bucket Lifecycle Configuration Quickstart Guide

Enable object lifecycle configuration on buckets to setup automatic deletion of objects after a specified number of days or a specified date.

> [Bucket Lifecycle Configuration Quickstart Guide](https://docs.min.io/docs/minio-bucket-lifecycle-guide.html)

### Select API Quickstart Guide

Traditional retrieval of objects is always as whole entities, i.e GetObject for a 5 GiB object, will always return
5 GiB of data. S3 Select API allows us to retrieve a subset of data by using simple SQL expressions. By using Select API
to retrieve only the data needed by the application, drastic performance improvements can be achieved.

> [Select API Quickstart Guide](https://docs.min.io/docs/minio-select-api-quickstart-guide.html)

### MinIO Gateway

* [MinIO Azure Gateway](https://docs.min.io/docs/minio-gateway-for-azure.html) - MinIO Gateway adds Amazon S3
compatibility to Microsoft Azure Blob Storage.

* [MinIO NAS Gateway](https://docs.min.io/docs/minio-gateway-for-nas.html) - MinIO Gateway adds Amazon S3 compatibility
to NAS storage. You may run multiple minio instances on the same shared NAS volume as a distributed object gateway.

* [MinIO S3 Gateway](https://docs.min.io/docs/minio-gateway-for-s3.html) - MinIO S3 Gateway adds MinIO features like
MinIO Browser and disk caching to AWS S3 or any other AWS S3 compatible service.

* [MinIO HDFS Gateway](https://docs.min.io/docs/minio-gateway-for-hdfs.html) - MinIO HDFS gateway adds Amazon S3 API
support to Hadoop HDFS filesystem. Applications can use both the S3 and file APIs concurrently without requiring
any data migration. Since the gateway is stateless and shared-nothing, you may elastically provision as many MinIO instances
as needed to distribute the load.

* [Disk Cache Quickstart Guide](https://docs.min.io/docs/minio-disk-cache-guide.html) - Disk caching feature here refers
to the use of caching disks to store content closer to the tenants. For instance, if you access an object from a lets
say gateway azure setup and download the object that gets cached, each subsequent request on the object gets served
directly from the cache drives until it expires.


## profile分析

### 安装Go环境

```shell script
# 下载地址：https://golang.google.cn/dl/
wget https://golang.google.cn/dl/go1.17.8.linux-amd64.tar.gz
# 解压
tar -zxf go1.17.8.linux-amd64.tar.gz
# 将 /usr/local/go/bin 目录添加至 PATH 环境变量
export PATH=$PATH:/usr/local/go/bin
```

### 安装 graphviz

安装图形可视化软件，用于生产分析图

```shell
yum install graphviz
```

### 分析 pprof

```shell
# 开始分析
mc admin profile start --type cpu,cpuio,mem,block,mutex,threads self
# 分析结束
mc admin profile stop self
# 分析解压后的 pprof 文件
go tool pprof -http=192.168.xxx.xxx:8081 profile-xxx-cpu.pprof
```

使用浏览器访问`http://192.168.xxx.xxx:8081`，查看资源使用情况。

* [Go性能分析大杀器PPROF](https://www.cnblogs.com/sy270321/p/12450151.html)
* [Hi, 使用多年的go pprof检查内存泄漏的方法居然是错的?!](https://colobu.com/2019/08/20/use-pprof-to-compare-go-memory-usage/)



## 参考

* [Github地址](https://github.com/minio/minio)
    * Github Doc
        * [英文](https://github.com/minio/minio/tree/master/docs)
        * [中文](https://github.com/minio/minio/tree/master/docs/zh_CN) - 仅作参考，不建议作为文档依据
* 官方文档
    * [英文](https://docs.min.io/)
    * [中文](https://docs.min.io/cn/minio-quickstart-guide.html) - 仅作参考，不建议作为文档依据

* CookBook
    * [Disaggregated HDP Spark and Hive with MinIO](https://docs.min.io/docs/disaggregated-spark-and-hadoop-hive-with-minio.html)
    * [S3cmd with MinIO Server](https://docs.min.io/docs/s3cmd-with-minio.html)
    * [AWS CLI with MinIO Server](https://docs.min.io/docs/aws-cli-with-minio.html)
    * [restic with MinIO Server](https://docs.min.io/docs/restic-with-minio.html)
    * [Store MySQL Backups in MinIO Server](https://docs.min.io/docs/store-mysql-backups-in-minio.html)
    * [Store MongoDB Backups in MinIO Server](https://docs.min.io/docs/store-mongodb-backups-in-minio.html)
    * [Store PostgreSQL Backups in MinIO Server](https://docs.min.io/docs/store-postgresql-backups-in-minio.html)
    * [Setup Caddy proxy with MinIO Server](https://docs.min.io/docs/setup-caddy-proxy-with-minio.html)
    * [Set up Nginx proxy with MinIO Server](https://docs.min.io/docs/setup-nginx-proxy-with-minio.html)
    * [Aggregate Apache Logs with fluentd plugin for MinIO](https://docs.min.io/docs/aggregate-apache-logs-with-fluentd-and-minio.html)
    * [Rclone with MinIO Server](https://docs.min.io/docs/rclone-with-minio-server.html)
    * [Setup Apache HTTP proxy with MinIO Server](https://docs.min.io/docs/setup-apache-http-proxy-with-minio-server.html)
    * [Upload Files Using Pre-signed URLs](https://docs.min.io/docs/upload-files-from-browser-using-pre-signed-urls.html)
    * [How to run MinIO in FreeNAS](https://docs.min.io/docs/how-to-run-minio-in-freenas.html)
    * [How to run multiple MinIO servers with Træfɪk](https://docs.min.io/docs/how-to-run-multiple-minio-servers-with-traef-k.html)
    * [How to use AWS SDK for Java with MinIO Server](https://docs.min.io/docs/how-to-use-aws-sdk-for-java-with-minio-server.html)
    * [How to use Paperclip with MinIO Server](https://docs.min.io/docs/how-to-use-paperclip-with-minio-server.html)

* examples
  * `PutObjectProgressBar` - [链接](https://github.com/minio/minio-java/blob/release/examples/PutObjectProgressBar.java)
  * `PutObjectUiProgressBar` - [链接](https://github.com/minio/minio-java/blob/release/examples/PutObjectUiProgressBar.java)


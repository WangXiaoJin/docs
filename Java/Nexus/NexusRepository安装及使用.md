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
    
## 文档

* [官方文档](https://help.sonatype.com/docs)



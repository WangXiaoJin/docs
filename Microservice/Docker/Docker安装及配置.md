## Docker安装及配置

###CentOS安装Docker

1. 前置条件

    * CentOS 7 64-bit
    * 卸载之前安装的Docker
        ```bash
        shell> sudo yum remove docker \
                               docker-common \
                               docker-selinux \
                               docker-engine
        ```
        卸载后所有的images、containers、volumes和networks都保留在`/var/lib/docker/`目录下。

2. repository安装

    **repository配置**
    
    * 安装需要的软件包，`yum-utils`提供了`yum-config-manager`工具包。`devicemapper`存储驱动依赖`device-mapper-persistent-data`、`lvm2`。
    
        ```bash
        shell> sudo yum install -y yum-utils device-mapper-persistent-data lvm2
        ```
    
    * 使用下面的命令来配置稳定版仓库
    
        ```bash
        shell> sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        ```
    
    * 可选步骤：启用`edge`和`test`仓库。`docker.repo`默认禁用这两个仓库，你可以通过下面的命令启用它们。
    
        ```bash
        shell> sudo yum-config-manager --enable docker-ce-edge
        shell> sudo yum-config-manager --enable docker-ce-test
        ```
    
    **安装Docker**
    
    * 更新`yum`的package index
    
        ```bash
        shell> sudo yum makecache fast
        ```
    
    * 安装最新版`Docker CE`
    
        ```bash
        shell> sudo yum install docker-ce
        ```
        > 注：如果你开启了多个Docker仓库，使用`yum install`/`yum update`命令 安装/更新 Docker时，总是安装所有仓库中最新版。
    
    * 在生产环境，你可能希望安装指定版本的Docker CE，而不是安装最新版。使用下面命令列出可安装的版本，`sort -r`使用版本号
    来排序所有可安装版本
        
        ```bash
        shell> yum list docker-ce.x86_64  --showduplicates | sort -r
        ```
        > 注：上述命令只显示二进制包，如果想显示源码包，省略后缀`.x86_64`。
        
        ```bash
        shell> sudo yum install docker-ce-<VERSION>
        ```
    
    * 运行Docker
    
        ```bash
        shell> sudo systemctl start docker
        ```
    
    * 验证Docker是否安装成功
        
        ```bash
        shell> sudo docker run hello-world
        ```

3. rpm包安装

    * 下载rpm包。下载地址：<https://download.docker.com/linux/centos/7/x86_64/stable/Packages/>。如果想下载非稳定版，
    把URL地址中`stable`换成`edge`。
    
    * 安装Docker CE
        ```bash
        shell> sudo yum install xxx.rpm
        ```
    
    * 启动Docker
        ```bash
        shell> sudo systemctl start docker
        ```
    
    * 验证Docker是否安装成功
        ```bash
        shell> sudo docker run hello-world
        ```

4. 卸载Docker

    * 卸载Docker
        ```bash
        shell> sudo yum remove docker-ce
        ```
    
    * 卸载Docker后`Images`、`containers`、`volumes`和自定义配置文件并不会自动删除，需要你自己删除。
        ```bash
        shell> sudo rm -rf /var/lib/docker
        ```

5. 开机自启动

    ```bash
    shell> sudo systemctl enable docker
    ```

6. 配置`Storage Driver` - [官方文档](https://docs.docker.com/engine/userguide/storagedriver/)

    官方推荐使用`overlay2`其次`overlay`：
    * `overlay2`时需要Linux内核4.0以上
    * 支持的文件系统格式
        * `ext4` (RHEL 7.1 only)
        * `xfs` (RHEL 7.2 and higher)，开启`d_type=true`，使用`xfs_info`验证`ftype`是否为`1`，请使用`-n ftype=1`格式化`xfs`文件系统
    * 搜索大量镜像文件时`overlay`性能高于`overlay2`，特别是镜像的层级比较多时。这与两者的架构设计有关，
    `overlay2`支持嵌套层级关系，而`overlay`则以文件硬链接的方式存在，所以`overlay`不需要一层一层去搜文件
    * `overlay`使用硬链接来共享文件减少磁盘空间，当镜像数量一多时，可能会报`too many links`错误。Ext4文件系统
    单文件最大硬链接数为65000，而`overlay2`不会有次问题
    * `open(2)`：OverlayFS并没有完全实现`POSIX standards`，例如`copy-up`功能：假如你的应用依次执行了
    `fd1=open("foo", O_RDONLY)`、`fd2=open("foo", O_RDWR)`。你此时期望`fd1`、`fd2`指向相同的文件，然而此时由于
    `copy-up`缘故，`fd2`指向一个新的文件（`upperdir`），`fd1`仍然指向镜像里的原始文件（`lowerdir`）。解决此问题
    可以在最开始执行`touch`指令直接生成一个新文件，然后再执行上述两个指令，此时他们会指向同一个文件。
    * 使用`yum`指令时最好确保`yum-plugin-ovl`已安装
    
    本机CentOS版本7.3，Docker版本17.06.1-ce。Docker默认使用`overlay`，当前版本下未大量测试`overlay2`，使用前请慎重。
    配置`overlay2`：
    
    * `XFS`文件系统请确保`ftype=1`开启
    * `yum install yum-plugin-ovl`
    * `systemctl stop docker`
    * 修改`/etc/docker/daemon.json`文件
        ```json
        {
          "storage-driver": "overlay2",
          "storage-opts": [
            "overlay2.override_kernel_check=true"
          ]
        }
        ```
    * 备份Docker文件：`mv /var/lib/docker /var/lib/docker-backup`
    * `systemctl start docker`
    * 验证Docker Storage Driver `docker info | grep Storage`

7. `Docker Swarm`

    * 开放端口
    
        在所有的节点上开放以下端口（根据自己需求而定）：
        * `2377 tcp`  用于集群管理的通信端口
            ```bash
            shell> firewall-cmd --add-port=2377/tcp
            shell> firewall-cmd --add-port=2377/tcp --permanent
            ```
        
        * `7946 tcp/udp`  各节点间通信使用
            ```bash
            shell> firewall-cmd --add-port=7946/tcp
            shell> firewall-cmd --add-port=7946/tcp --permanent
  
            shell> firewall-cmd --add-port=7946/udp
            shell> firewall-cmd --add-port=7946/udp --permanent
            ```
        
        * `4789 udp`  for overlay network traffic
            ```bash
            shell> firewall-cmd --add-port=4789/udp
            shell> firewall-cmd --add-port=4789/udp --permanent
            ```
        
        * 如果你新建了一个开启`encryption`功能(`--opt encrypted`)的`overlay network` ，确保`50 (ESP)`协议可以访问
        
    * 初始化`swarm`
        ```bash
        shell> docker swarm init --advertise-addr 192.168.206.137
        ```
        初始化swarm，指定`192.168.206.137`为`manager node`通信地址
        
    * 增加节点（`manager`|`worker`）
        
        `manager node`作用：
        * 维护集群状态
        * 调度service
        * 提供`swarm mode HTTP API`
        
        > 注：Docker `manager node` 自带容灾功能，建议部署**奇数**（`1`除外）个`manager node`，官方建议部署`7`个。
        不是部署的越多性能越高，也不是部署的越少越好。
        > 1) 部署三个`manager node` ，最大允许一个`manager node` 出现故障
        > 2) 部署五个`manager node` ，最大允许两个`manager node` 出现故障
        > 3) 部署`N`个`manager node` ，最大允许`(N-1)/2`个`manager node` 出现故障
        
        增加`worker`节点。在`manager node`机子使用以下命令查看token：
        ```bash
        shell> docker swarm join-token worker
        To add a worker to this swarm, run the following command:
          
          docker swarm join --token SWMTKN-1-30fgdzfpjm0wq5gtcvtlhm4s0waehhlyverbh8xml8b4m8gx57-012e9t9f9cslg2p8cupdfi72h 192.168.206.137:2377
        ```
        
        增加`manager`节点。在`manager node`机子使用以下命令查看token：
        ```bash
        shell> docker swarm join-token manager
        To add a manager to this swarm, run the following command:
          
          docker swarm join --token SWMTKN-1-30fgdzfpjm0wq5gtcvtlhm4s0waehhlyverbh8xml8b4m8gx57-6xhjkxfu0a1pkafxahgh2dpcx 192.168.206.137:2377
        ```
        
        增加`manager`、`worker`节点唯一不同的就是`--token`值不同。
        
        在`192.168.206.138`、`192.168.206.139`机器上各执行一次增加worker节点命令：
        ```bash
        shell> docker swarm join --token SWMTKN-1-30fgdzfpjm0wq5gtcvtlhm4s0waehhlyverbh8xml8b4m8gx57-012e9t9f9cslg2p8cupdfi72h 192.168.206.137:2377
        ```
        此时`192.168.206.137`为`manager node`，`192.168.206.138`、`192.168.206.139`为`worker node`。
        可以通过`docker info`指令查看`swarm`状态。
    
        > 注：我纯粹为了测试使用，只设置了一个`manager node`，生产环境千万不能只有一台`manager node`。默认配置下，
        manager node会被分配tasks，如果你不想manager node运行container，设置`--availability drain`（生产环境建议设置），
        此时调度会停止此节点上的tasks，转移到其他`Active node`上运行tasks，且不会分配新的tasks到此节点上。
        通过`docker node update`来修改`--availability`。  
        另外可以通过`docker node promote`、`docker node demote`来切换`worker/manager`角色。
    
    * 部署service
    
        ```bash
        shell> docker service create -d --replicas 3 -p 8080:8070 --name test registry.cn-hangzhou.aliyuncs.com/test-ns/test:0.1
        ```
        * `--name` service名
        * `--replicas` 运行的实例数
        * `--mode global` 全局模式，默认为`replicated`。即每个node都会运行一个task，使用此参数后不需要设置`--replicas`。
        * `-p 8080:8070` 对外端口映射。`8080`对外暴露端口，`8070`容器暴露端口。可以在后面增加协议类型，默认为`tcp`，
        如：`-p 8080:8070/tcp`、`-p 8080:8070/udp`，两种类型都使用时必须两者都写。
        
        查看service状态：
        ```bash
        shell> docker service inspect test
        shell> docker service ps test
        ```
        
    * service扩容/缩容
    
        ```bash
        # 缩容至两个实例
        shell> docker service scale test=2
        ```
        
    * `docker-compose使用`
    
        [Compose file语法 - 重要](https://docs.docker.com/compose/compose-file/)
        
        [Control startup order in Compose - 重要](https://docs.docker.com/compose/startup-order/)
        * [wait-for-it](https://github.com/vishnubob/wait-for-it) - 通过脚本控制服务启动顺序，`host:port`作为判断依赖
        * [dockerize](https://github.com/jwilder/dockerize) 
            1. 提供配置文件模板化功能
            2. 把日志文件重定向到`Docker Container`的`STDOUT`/`STDERR`，方便使用`docker logs`功能
            3. 控制服务启动顺序
        * [wait-for](https://github.com/Eficode/wait-for) - 类似`wait-for-it`
        
        [Networking in Compose - 重要](https://docs.docker.com/compose/networking/)
        
        [Extend services in Compose](https://docs.docker.com/compose/extends/)
    
        例子：
        * [创建Services](https://docs.docker.com/get-started/part3/)
        * [Stacks增加Services](https://docs.docker.com/get-started/part5/)
        
        > 注：`docker-compose.yml`文件后缀可以为`.yml`、`.yaml`,文件随随意定义

8. 安装`Docker Machine`（可选）

    * 安装`command completion` - 参考[Docker-compose安装及使用](Docker-compose安装及使用.md)安装`command completion`
    * 安装`Docker Machine`，[Github下载地址](https://github.com/docker/machine/releases/)
        ```bash
        shell> curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
               chmod +x /tmp/docker-machine &&
               sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
        ```
    * 安装`docker-machine bash completion`脚本
        ```bash
        shell> scripts=( docker-machine-prompt.bash docker-machine-wrapper.bash docker-machine.bash ); for i in "${scripts[@]}"; do sudo wget https://raw.githubusercontent.com/docker/machine/v0.13.0/contrib/completion/bash/${i} -P /etc/bash_completion.d; done
        ```
        上述三个脚本特性：
        * command completion
        * a function that displays the active machine in your shell prompt
        * a function wrapper that adds a `docker-machine use` subcommand to switch the active machine
        
        To enable the `docker-machine` shell prompt, add `$(__docker_machine_ps1)` to your `PS1` setting in `~/.bashrc`：
        ``bash
        PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '
        ```
    * 卸载Docker Machine
        ```bash
        shell> rm $(which docker-machine)
        ```
    
    > `Docker Machine`用来创建、管理、使用`本地或远程Docker host`  
      [官方文档](https://docs.docker.com/machine/install-machine/)


9. 常见问题

    * 在内核低于`3.10`的系统或缺失某些模块时，Docker将不能正常运行。检查内核兼容性，通过以下脚本(仅适用于Linux系统)：
        ```bash
        shell> curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh
        shell> bash ./check-config.sh
        ```
    * Cannot connect to the Docker daemon
    
        如果你碰到了下面的异常提示信息，你的Docker客户端连接的`Docker daemon` host可能配错了或者不通
        ```
        Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?
        ```
        
        请通过以下命令检查你的Docker客户端的配置是否有错：
        ```bash
        shell> env | grep DOCKER_HOST
        ```
        
        如果你配置了`DOCKER_HOST`环境变量，则Docker client连接Docker daemon时使用此配置的host。没配则使用local host。
        删除环境变量命令：
        ```bash
        shell> unset DOCKER_HOST
        ```
        
        > 你可以把环境变量配置在`~/.bashrc`或`~/.profile`文件中
    
    * swarm service 状态一直处于 `pending`
    
        * 当所有的node都处于`pause`或`drain`状态时，你创建了一个新的service，此时将会一直处于`pending`状态，
        直到有可用的node为止。第一个可用的node将会接收所有的tasks。
        
        * 创建service时你指定了需要多大的内存，如果swarm中没有一个node能提供足够大的内容供你创建service，
        则一直处于`pending`状态，直到有node能提供你所需的内存时，才会在此node上分配tasks。
        
        * 创建service时你配置的约束条件，而此时没有一个node能够满足此条件，则一直处于`pending`状态。
    
    * IP转发问题
    
        如果你通过`systemd-network`手动配置网络，且`systemd`版本为219以上，Docker容器可能无法访问你的网络。
        `systemd`从220版本开始，网络转发配置（`net.ipv4.conf.<interface>.forwarding`）默认关闭，这个配置阻止IP转发，
        且于Docker容器的`net.ipv4.conf.all.forwarding`配置相冲突。为了在 RHEL、CentOS、Fedora解决此问题，
        编辑`/usr/lib/systemd/network/`目录下的`<interface>.network`文件（例：`/usr/lib/systemd/network/80-container-host0.network`）
        添加下面的配置：
        ```
        [Network]
        ...
        IPForward=kernel
        # OR
        IPForward=true
        ...
        ```
        上述配置项允许容器进行IP转发
    
    > 参考文档：[Optional Linux post-installation steps【重要】](https://docs.docker.com/engine/installation/linux/linux-postinstall/)

9. 文档
    
    * [Get started](https://docs.docker.com/get-started/part2/)
    * [Daemon CLI reference (dockerd)](https://docs.docker.com/engine/reference/commandline/dockerd/)
    * Dockerfile
        * [Dockerfiles文档 - 【重要】](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
        * [Dockerfile reference - 【重要】](https://docs.docker.com/engine/reference/builder/)
        
            `docker build`命令跑在`docker daemon`中，而不是`docker CLI`。`build`进程首先会把`build context`目录下所有的
            文件发送给`docker daemon`，所以你应该尽可能的减少`build context`里面的文件。千万不要使用根路径作(`/`)为
            `build context`，否则会把所有的系统文件一起发送给`docker daemon`。
            
            为了增加构建性能，可以在`build context`目录下增加`.dockerignore`文件来排除指定的文件或目录。
    * [Create a base image](https://docs.docker.com/engine/userguide/eng-image/baseimages/)
    * [multi-stage builds](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)
    * [Exposing and publishing ports](https://docs.docker.com/engine/userguide/networking/#exposing-and-publishing-ports)
    * [Docker and iptables](https://docs.docker.com/engine/userguide/networking/#docker-and-iptables)
    * [`docker network` commands](https://docs.docker.com/engine/userguide/networking/work-with-networks/)
    * [Swarm mode overlay network security model](https://docs.docker.com/engine/userguide/networking/overlay-security-model/)
    * [Labels](https://docs.docker.com/engine/userguide/labels-custom-metadata/)
    * [Configure and troubleshoot the Docker daemon](https://docs.docker.com/engine/admin/)
    * [Start containers automatically](https://docs.docker.com/engine/admin/start-containers-automatically/)
    * [Limit a container's resources - 【重要】](https://docs.docker.com/engine/admin/resource_constraints/)
    * [Prune unused Docker objects - 【重要】](https://docs.docker.com/engine/admin/pruning/)
    * [Format command and log output - 【重要】](https://docs.docker.com/engine/admin/formatting/)
    * [Manage data in Docker(Volumes/Bind mounts/tmpfs mounts) - 【重要】](https://docs.docker.com/engine/admin/volumes/)  
        `-v`和`--mount`区别：在使用`bind mount`时，如果docker宿主机没有对应的`src`路径，`-v`会自动创建一个空的对应文件夹，
        而`--mount`不会自动创建`src`文件夹，且会抛出异常。`dst`配置路径（容器里的路径）使用会自动创建。
        
        `bind mount`：即使容器中bind-mount的目录为非空目录，docker宿主机的目录会直接覆盖容器目录的内容，即容器中目录内容始终会被影藏。
        
        `volume`：和`bind mount`有点区别，如果`volume`的`src`目录有内容则使用`src`目录里的内容并影藏容器目录的内容。
        如果`src`目录为空，则使用容器目录的内容，且容器目录中内容能在`src`目录中看到。
    * 【Swarm】
        * [Apply rolling updates to a service](https://docs.docker.com/engine/swarm/swarm-tutorial/rolling-update/)  
        * [Use swarm mode routing mesh - 【重要】](https://docs.docker.com/engine/swarm/ingress/)  
        * [Manage nodes in a swarm - 【重要】](https://docs.docker.com/engine/swarm/manage-nodes/)  
        * [How nodes work - 【重要】](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)  、
            [How services work - 【重要】](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
        * [Deploy services to a swarm - 【非常重要】](https://docs.docker.com/engine/swarm/services/)  
        * [Store service configuration data](https://docs.docker.com/engine/swarm/configs/)  
        * [Manage sensitive data with Docker secrets](https://docs.docker.com/engine/swarm/secrets/)  
        * [Swarm administration guide - 【重要】](https://docs.docker.com/engine/swarm/admin_guide/)  
    * [Docker command line - 命令行使用规则](https://docs.docker.com/engine/reference/commandline/cli/)  
    * [Docker-in-Docker](https://blog.docker.com/2013/09/docker-can-now-run-within-docker/)  

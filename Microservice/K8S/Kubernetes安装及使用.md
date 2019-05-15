## Kubernetes安装及使用

### 安装`kubeadm`

1. 准备工作
    * 支持以下系统
        * Ubuntu 16.04+
        * Debian 9
        * CentOS 7
        * RHEL 7
        * Fedora 25/26 (best-effort)
        * HypriotOS v1.0.1+
        * Container Linux (tested with 1576.4.0)
    * 每台机子需要2GB或以上内存
    * 2核或以上CPU
    * 集群中所有机器间的网络必须通畅

2. 系统配置

    * 确保每个节点的`hostname`/`MAC`/`product_uuid`必须唯一，否则会安装失败
        * 可通过`ip link`、`ifconfig -a`命令来获取`MAC`地址
        * 通过`sudo cat /sys/class/dmi/id/product_uuid`命令来获取`product_uuid`
    
    * 机器列表及配置hostname
        
        | Hostname | IP | Type |
        | ----| ---- | ---- |
        | k8s-master-1  | 192.168.206.137  | Master node |
        | k8s-master-2  | 192.168.206.138  | Master node |
        | k8s-master-3  | 192.168.206.139  | Master node |
        | k8s-node-1  | 192.168.206.141  | Worker node |
        | k8s-node-2  | 192.168.206.142  | Worker node |
        
        ```bash
        shell> hostnamectl set-hostname k8s-master-1  #修改hostname
        shell> vim /etc/hosts
        192.168.206.137 k8s-master-1    #【例子】IP:192.168.206.137  hostname:k8s-master-1
        shell> reboot
        ```
        
        > 注：在所有机器上执行上述命令，IP和hostname修改成各自的。
    
    * 在所有节点上增加kubernetes仓库
        
        ```bash
        shell> cat <<EOF > /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        EOF
        shell> yum clean all
        shell> yum makecache
        ```
        
        **注：因Google被墙，可以换成阿里地址：**
        ```bash
        #kubernetes yum源
        shell> cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
        [kubernetes]
        name=Kubernetes
        baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
        enabled=1
        gpgcheck=0
        EOF
        shell> yum clean all
        shell> yum makecache
        ```
        Docker替代仓库（可选）：
        ```bash
        #docker yum源
        shell> cat >> /etc/yum.repos.d/docker.repo <<EOF
        [docker-repo]
        name=Docker Repository
        baseurl=http://mirrors.aliyun.com/docker-engine/yum/repo/main/centos/7
        enabled=1
        gpgcheck=0
        EOF
        shell> yum clean all
        shell> yum makecache
        ```
        
    * 在所有kubernetes节点上进行系统更新（可选）
    
        ```bash
        shell> yum update -y
        ```
    
    * 在所有节点上设置SELINUX为permissive模式
    
        ```bash
        shell> vi /etc/selinux/config
        SELINUX=permissive
        shell> setenforce 0   #setenforce 0 设置SELinux为permissive模式，setenforce 1 设置SELinux为enforcing模式
        shell> getenforce #验证是否配置成功
        ```
    
    * 在所有节点上配置`net.bridge.bridge-nf-call-iptables`
        
        ```bash
        shell> cat <<EOF >  /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        EOF
        shell> sysctl --system
        ```
        
    * 在所有kubernetes节点上禁用swap，否则`kubelet`将不能正常工作。
    
        Kubernetes 1.8开始要求关闭系统的Swap，如果不关闭，默认配置下kubelet将无法启动。可以通过kubelet的启动参数
        `--fail-swap-on=false`更改这个限制。关闭Swap方法：
        
        ```
        1. swapoff -a
        2. 修改/etc/fstab文件，注释掉 SWAP 的自动挂载，在Swap那行前加上`#`
        3. 使用free -m 命令确认是否禁用成功
        ```
    
    * 重启所有节点
    
        ```bash
        shell> reboot
        ```

3. 安装Docker [参考文档](../Docker/Docker安装及配置.md)

4. 安装`kubeadm`、`kubelet` 、`kubectl`

    ```bash
    # 查看可安装kubeadm版本
    shell> yum list kubeadm --showduplicates
 
    # 注：kubectl 不需要每个节点上都安装，可在Master上安装
    shell> yum install -y kubelet kubeadm kubectl
 
    # 启动之前配置 kubelet的cgroup driver，参考下面的文档
    shell> systemctl enable kubelet && systemctl start kubelet
    ```
    
    注意事项：
    * 安装过程中出现以下错误时，请参考[RabbitMQ安装socat](../../MQ/RabbitMQ/RabbitMQ安装及配置.md)
        ```
        Error downloading packages:
          socat-1.7.3.2-2.el7.x86_64: [Errno 256] No more mirrors to try.
        ```
        
    * 配置`kubelet` 的`cgroup driver`。确保`kubelet`和`Docker`的`cgroup driver`一样：
        ```bash
        shell> docker info | grep -i cgroup
        shell> cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
        ```
        
        如果`cgroup driver`配置不同，使用下面命令来修改配置：
        ```bash
        shell> sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
        ```
        如果配置文件中没有此配置项，则手动加上上面的配置项
        
        重启`kubelet`：
        ```bash
        shell> systemctl daemon-reload
        shell> systemctl restart kubelet
        ```
    
5. 需开放的端口

    **Master node(s)**
    
    | Protocol | Direction | Port Range | Purpose |
    | ----| ---- | ---- | ---- |
    | TCP  | Inbound  | 6443（可配置） | Kubernetes API server |
    | TCP  | Inbound  | 2379-2380 | etcd |
    | TCP  | Inbound  | 10250 | Kubelet API |
    | TCP  | Inbound  | 10251 | kube-scheduler |
    | TCP  | Inbound  | 10252 | kube-controller-manager |
    | TCP  | Inbound  | 10255 | Kubernetes API server |
    
    **Worker node(s)**
    
    | Protocol | Direction | Port Range | Purpose |
    | ----| ---- | ---- | ---- |
    | TCP  | Inbound  | 10250 | Kubelet API |
    | TCP  | Inbound  | 10255 | Read-only Kubelet API |
    | TCP  | Inbound  | 30000-32767（默认值） | NodePort Services |
    
6. 安装`etcd`

    我们在三台机子`k8s-master-1`、`k8s-master-2`、`k8s-master-3`上各安装一个etcd实例 

    * 安装`cfssl`、`cfssljson`
    
        ```bash
        shell> curl -o /usr/local/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
        shell> curl -o /usr/local/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
        shell> chmod +x /usr/local/bin/cfssl*
        ```
    
    * 在`k8s-master-1`机子上生成CA证书
    
        ```bash
        shell> mkdir -p /etc/kubernetes/pki/etcd
        shell> cd /etc/kubernetes/pki/etcd
        ```
        
        ```bash
        cat >ca-config.json <<EOF
        {
            "signing": {
                "default": {
                    "expiry": "43800h"
                },
                "profiles": {
                    "server": {
                        "expiry": "43800h",
                        "usages": [
                            "signing",
                            "key encipherment",
                            "server auth",
                            "client auth"
                        ]
                    },
                    "client": {
                        "expiry": "43800h",
                        "usages": [
                            "signing",
                            "key encipherment",
                            "client auth"
                        ]
                    },
                    "peer": {
                        "expiry": "43800h",
                        "usages": [
                            "signing",
                            "key encipherment",
                            "server auth",
                            "client auth"
                        ]
                    }
                }
            }
        }
        EOF
        ```
        
        ```bash
        cat >ca-csr.json <<EOF
        {
            "CN": "etcd",
            "key": {
                "algo": "rsa",
                "size": 2048
            }
        }
        EOF
        ```
        
        ```bash
        shell> cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
        ```
    
    * 在`k8s-master-1`机子上生成client证书
    
        ```bash
        cat >client.json <<EOF
        {
            "CN": "client",
            "key": {
                "algo": "ecdsa",
                "size": 256
            }
        }
        EOF
        ```
        
        ```bash
        shell> cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client.json | cfssljson -bare client
        ```
        
        此命令会生成`client.pem`、`client-key.pem`
        
    * 从`k8s-master-1`机子上拷贝已生成的证书至`k8s-master-2`、`k8s-master-3`
    
        ```bash
        shell> mkdir -p /etc/kubernetes/pki/etcd
        shell> cd /etc/kubernetes/pki/etcd
        shell> scp root@<k8s-master-1-ip-address>:/etc/kubernetes/pki/etcd/ca.pem .
        shell> scp root@<k8s-master-1-ip-address>:/etc/kubernetes/pki/etcd/ca-key.pem .
        shell> scp root@<k8s-master-1-ip-address>:/etc/kubernetes/pki/etcd/client.pem .
        shell> scp root@<k8s-master-1-ip-address>:/etc/kubernetes/pki/etcd/client-key.pem .
        shell> scp root@<k8s-master-1-ip-address>:/etc/kubernetes/pki/etcd/ca-config.json .
        ```
    
    * 在`k8s-master-1`、`k8s-master-2`、`k8s-master-3`三台机子上执行以下命令，生成`peer.pem`、`peer-key.pem`、`server.pem`、`server-key.pem`
    
        ```bash
        shell> export PEER_NAME=$(hostname)
        # 如果没有eth1则选择合适的网络接口
        shell> export PRIVATE_IP=$(ip addr show eth1 | grep -Po 'inet \K[\d.]+')
        # 验证 PRIVATE_IP PEER_NAME
        shell> echo $PEER_NAME $PRIVATE_IP
        shell> cfssl print-defaults csr > config.json
        shell> sed -i '0,/CN/{s/example\.net/'"$PEER_NAME"'/}' config.json
        shell> sed -i 's/www\.example\.net/'"$PRIVATE_IP"'/' config.json
        shell> sed -i 's/example\.net/'"$PEER_NAME"'/' config.json
        shell> cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server config.json | cfssljson -bare server
        shell> cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer config.json | cfssljson -bare peer
        ```
    
    * 下载`etcd`
    
        ```bash
        shell> export ETCD_VERSION=v3.1.12
        shell> curl -sSL https://github.com/coreos/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz | tar -xzv --strip-components=1 -C /usr/local/bin/
        shell> rm -rf etcd-$ETCD_VERSION-linux-amd64*
        ```
        
        > 注：`Kubernetes v1.9`首选`etcd v3.1.10`，`Kubernetes v1.10`首选`etcd v3.1.12`。如果系统已默认安装了`etcd`，则先卸载它。
    
    * 生成etcd环境变量文件
    
        ```bash
        shell> touch /etc/etcd.env
        # 验证 PRIVATE_IP PEER_NAME
        shell> echo $PEER_NAME $PRIVATE_IP
        shell> echo "PEER_NAME=$PEER_NAME" >> /etc/etcd.env
        shell> echo "PRIVATE_IP=$PRIVATE_IP" >> /etc/etcd.env
        ```
    
    * 配置etcd systemd unit file
    
        ```bash
        shell> cat >/etc/systemd/system/etcd.service <<EOF
        [Unit]
        Description=etcd
        Documentation=https://github.com/coreos/etcd
        Conflicts=etcd.service
        Conflicts=etcd2.service
        
        [Service]
        EnvironmentFile=/etc/etcd.env
        Type=notify
        Restart=always
        RestartSec=5s
        LimitNOFILE=40000
        TimeoutStartSec=0
        
        ExecStart=/usr/local/bin/etcd --name ${PEER_NAME} \
            --data-dir /var/lib/etcd \
            --listen-client-urls https://${PRIVATE_IP}:2379 \
            --advertise-client-urls https://${PRIVATE_IP}:2379 \
            --listen-peer-urls https://${PRIVATE_IP}:2380 \
            --initial-advertise-peer-urls https://${PRIVATE_IP}:2380 \
            --cert-file=/etc/kubernetes/pki/etcd/server.pem \
            --key-file=/etc/kubernetes/pki/etcd/server-key.pem \
            --client-cert-auth \
            --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
            --peer-cert-file=/etc/kubernetes/pki/etcd/peer.pem \
            --peer-key-file=/etc/kubernetes/pki/etcd/peer-key.pem \
            --peer-client-cert-auth \
            --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
            --initial-cluster k8s-master-1=https://192.168.206.137:2380,k8s-master-2=https://192.168.206.138:2380,k8s-master-3=https://192.168.206.139:2380 \
            --initial-cluster-token my-etcd-token \
            --initial-cluster-state new
        
        [Install]
        WantedBy=multi-user.target
        EOF
        ```
        
    * 开放etcd端口
        
        ```bash
        shell> firewall-cmd --add-port=2379/tcp --permanent
        shell> firewall-cmd --add-port=2380/tcp --permanent
        shell> firewall-cmd --reload
        ```
    
    * 启动etcd
    
        ```bash
        shell> systemctl daemon-reload
        # 设置为开机启动
        shell> systemctl enable etcd
        shell> systemctl start etcd
        shell> systemctl status etcd
        ```

7. 配置集群
    
    * 初始化master
    
        创建`kubeadm init`的配置文件：
        ```bash
        cat >k8s-config.yaml <<EOF
        apiVersion: kubeadm.k8s.io/v1alpha1
        kind: MasterConfiguration
        kubernetesVersion: v1.10.0 #可自行选择需要安装的版本
        api:
          # 本机IP
          advertiseAddress: 192.168.206.137
        etcd:
          endpoints:
          - https://192.168.206.137:2379
          - https://192.168.206.138:2379
          - https://192.168.206.139:2379
          caFile: /etc/kubernetes/pki/etcd/ca.pem
          certFile: /etc/kubernetes/pki/etcd/client.pem
          keyFile: /etc/kubernetes/pki/etcd/client-key.pem
        networking:
          dnsDomain: "cluster.local"
          # flannel配置
          podSubnet: 10.244.0.0/16
        apiServerCertSANs:
        - 127.0.0.1
        - 192.168.206.137 #本机IP
        - 192.168.206.140 #通过负载均衡器来暴露API Service，可以是IP/域名（Nginx/Keepalived/HAproxy）
        apiServerExtraArgs:
          endpoint-reconciler-type: "lease"
        # 因国内不能访问gcr.io/google_containers，所以替换成其他的
        imageRepository: "anjia0532"
        EOF
        ```
        
        如果你不能访问`k8s.gcr.io/pause-amd64`镜像，需要自定义`--pod-infra-container-image`参数时，执行下面命令：
        ```bash
         # 查看当前kubelet使用的pause-amd64镜像版本
        shell> kubelet -h | grep pause
          --pod-infra-container-image string   The image whose network/ipc namespaces containers in each pod will use. (default "k8s.gcr.io/pause-amd64:3.1")
        # 这里我们替换成anjia0532/pause-amd64镜像
        shell> docker pull anjia0532/pause-amd64:3.1
  
        # 修改kubelet的pod-infra-container-image参数，增加下面的配置项
        shell> vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
          Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=anjia0532/pause-amd64:3.1"
        
        shell> systemctl daemon-reload
        shell> systemctl restart kubelet
        ```
        
        ```bash
        shell> kubeadm init --config=k8s-config.yaml
          kubeadm join 192.168.206.137:6443 --token c1s6nw.ti6y77dxyptys9ql --discovery-token-ca-cert-hash sha256:e4fe3af6087bd33b0a95cdf519af0064bf9239ef7922e0934dc98991eba02114
        ```
        > 注：配置文件初始化为alpha特性，以后可能会有所改动，执行前请和官方文档对照。如果想自定义镜像时，
        你可能不知道镜像使用哪个版本，执行完上面`kubeadm init --config=k8s-config.yaml`会生成对应yaml文件，
        进入`/etc/kubernetes/manifests/`目录下找对应的镜像版本。
        
        如果执行`kubeadm init`出错了，想还原状态则执行下面命令：
        ```bash
        # 备份已生成的证书
        shell> mv /etc/kubernetes/pki /etc/kubernetes/pki-back
        # 重置状态
        shell> kubeadm reset
        # 还原证书
        shell> mv /etc/kubernetes/pki-back /etc/kubernetes/pki
        ```
        
        * 因国内不能访问`gcr.io/google-containers`镜像，可使用以下两种方式之一：
            1. `gcr.io/google-containers`替换成`anjia0532`，[参考地址](https://github.com/anjia0532/gcr.io_mirror)
                ```bash
                shell> docker pull gcr.io/google-containers/federation-controller-manager-arm64:v1.3.1-beta.1
                shell> docker pull anjia0532/federation-controller-manager-arm64:v1.3.1-beta.1
                ```
            2. 通过Github和Docker Hub来自动构建镜像，[参考地址](https://blog.csdn.net/CHENYUFENG1991/article/details/79118330)
        
        > 注：[kubeadm init官方文档 - 重要](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)
        ，含`kubeadm init`执行流程。
    
    * 如果你想让非root用户操作`kubectl`，你需要运行下面这些命令：
        ```bash
        shell> mkdir -p $HOME/.kube
        shell> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        shell> sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```
        如果你使用root操作，你需要运行下面命令：
        ```bash
        shell> vi ~/.bashrc
          export KUBECONFIG=/etc/kubernetes/admin.conf
        shell> source ~/.bashrc
        ```
        
    * Installing a pod network
        
        You MUST install a pod network add-on so that your pods can communicate with each other.
        
        The network must be deployed before any applications. Also, kube-dns, an internal helper service, will not start up before a network is installed. kubeadm only supports Container Network Interface (CNI) based networks (and does not support kubenet).
        
        当前我选用`flannel`，执行`kubeadm init`时需要传参数`--pod-network-cidr=10.244.0.0/16`。`flannel`支持`amd64`、`arm`、
        `arm64`、`ppc64le`，当你运行在非`amd64`环境中，你需要手动修改yaml配置文件内容，替换掉`amd64`相关配置。
        
        ```bash
        shell> kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
        ```
        
        一旦网络插件安装好，你可以通过`kubectl get pods --all-namespaces`查看`kube-dns`是否正在运行，从而确定网络查看是否运行正常。
        当`kube-dns`运行成功后，你可以继续添加node。
    
    * 让Master节点部署Pod（生成环境不建议使用）
        
        默认情况下Master节点不会部署Pod，如果你想去除此限制，执行以下命令：
        ```bash
        shell> kubectl taint nodes --all node-role.kubernetes.io/master-
        ```
        此命令会从所有节点中删除`node-role.kubernetes.io/master` taint，包括Master节点。
    
    * 增加Nodes
        
        以root用户（`sudo su -`）执行：
        ```bash
        shell> kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
        ```
        此命令可在`kubeadm init`输出内容中找到。如果你想想使用IPv6，则IPv6需用方括号括起来，如：[fd00::101]:2073

    * 从非Master机器上操作K8S集群
    
        ```bash
        shell> scp root@<master ip>:/etc/kubernetes/admin.conf .
        shell> kubectl --kubeconfig ./admin.conf get nodes
        ```

7. 删除节点

    在Master节点执行：
    ```bash
    shell> kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
    shell> kubectl delete node <node name>
    ```
    在被删节点机子上执行：
    ```bash
    shell> kubeadm reset
 
    #删除已配置的网络需执行以下命令
    shell> ifconfig cni0 down
    shell> ip link delete cni0
    shell> ifconfig flannel.1 down
    shell> ip link delete flannel.1
    shell> rm -rf /var/lib/cni/
    ```
    
8. 参考文档
    * [基于kubeadm的kubernetes高可用集群部署](https://github.com/cookeem/kubeadm-ha/blob/master/README_CN.md)
    * [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
    * [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
    * [kubeadm tools reference](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)
    * [Troubleshooting kubeadm](https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/)
    * [Kubeadm实现原理](https://kubernetes.io/docs/reference/setup-tools/kubeadm/implementation-details/)
    * [Kubeadm设计文档 - 【重要】](https://github.com/kubernetes/kubeadm/blob/master/docs/design/design_v1.10.md)
    * [kubectl命令详解 - 官方文档](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
        
    

### 安装`kubectl`

1. 下载最新版软件
    
    ```bash
    shell> curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    ```
    
    下载指定版本的kubectl，请替换上述命令中的`$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`
    
    例如下载`v1.9.0`：`curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl`

2. 使`kubectl`变成可执行文件

    ```bash
    shell> chmod +x ./kubectl
    ```

3. Move the binary in to your PATH
    
    ```bash
    shell> sudo mv ./kubectl /usr/local/bin/kubectl
    ```

4. 确认`kubectl`是否配置正确
    
    ```bash
    shell> kubectl cluster-info
    ```
    
    如果上述命令有响应，则表明kubectl连接集群的配置正确。如果返回下面的错误信息，表明kubectl配置错误或不能连接到K8S集群：
    ```
    The connection to the server <server-name:port> was refused - did you specify the right host or port?
    ```
    
    如果你不能访问K8S集群，可通过下面命令查看配置：
    ```bash
    shell> kubectl cluster-info dump
    ```

5. Enabling shell autocompletion

    * Using bash
    
        * To add kubectl autocompletion to your current shell, run `source <(kubectl completion bash)`
        * To add kubectl autocompletion to your profile, so it is automatically loaded in future shells run:
            ```bash
            shell> echo "source <(kubectl completion bash)" >> ~/.bashrc
            ```
    
    * Using Zsh
    
        If you are using zsh edit the ~/.zshrc file and add the following code to enable kubectl autocompletion:
        ```
        if [ $commands[kubectl] ]; then
          source <(kubectl completion zsh)
        fi
        ```

### 安装`ingress`【可选】 - 待完善
    
* [官方文档](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [Github地址](https://github.com/kubernetes/ingress-nginx)

### 安装`telepresence` - [官方文档](https://www.telepresence.io/discussion/overview) 【可选】

只适用于Linux、Mac，用于在本机调试应用，本地应用能与远程K8S集群里的应用通信，本地应用就像远程K8S集群的一部分。

### 监控

* [Tools for Monitoring Compute, Storage, and Network Resources](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
* [Controller manager metrics](https://kubernetes.io/docs/concepts/cluster-administration/controller-metrics/) : http://{MasterIP}:10252/metrics
* [Node Problem Detector](https://kubernetes.io/docs/tasks/debug-application-cluster/monitor-node-health/) - 
      监控Node运行期间出现的系统问题（如：CPU、内核、磁盘等）

### TLS

* [Certificates - 使用`easyrsa`、`openssl`、`cfssl`创建证书](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)
* [Certificate Rotation](https://kubernetes.io/docs/tasks/tls/certificate-rotation/)
* [Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) : http://{MasterIP}:10252/metrics
* [K8S集成Nginx例子 - 【重要】](https://github.com/kubernetes/examples/tree/master/staging/https-nginx/) - 
    创建Nginx Service例子，使用TLS、secrets、configmap、Nginx自动重启（监控Nginx配置文件）等功能

### helm - K8S包管理工具，方便在K8S集群中部署软件 - [官方文档](https://docs.helm.sh/) 【可选】


### 使用及文档

1. 常用

    * [K8S各种使用例子](https://github.com/kubernetes/examples/tree/master/staging)
    
    * Migrating from imperative commands to imperative object configuration
        
        Migrating from imperative commands to imperative object configuration involves several manual steps.
        
        1. Export the live object to a local object configuration file:
        
            ```bash
            kubectl get <kind>/<name> -o yaml --export > <kind>_<name>.yaml
            ```
            
        2. Manually remove the status field from the object configuration file.
        
        3. For subsequent object management, use `replace` exclusively.
        
            ```bash
            kubectl replace -f <kind>_<name>.yaml
            ```

2. 概念 - 架构
    
    * [Nodes - 【重要】](https://kubernetes.io/docs/concepts/architecture/nodes/)
    * [Images](https://kubernetes.io/docs/concepts/containers/images/)
       ：K8S加载私有仓库镜像配置
    * [Pod解析](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
       ：此链接的同级目录都比较重要（包含POD生命周期等）
    * [Container Lifecycle Hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)
       ：容器的生命周期及钩子实现
    * [Names](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/)
    * [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
    * [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
    * [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
    * [Federation - 【重要】](https://kubernetes.io/docs/concepts/cluster-administration/federation/)
       ：多集群管理

3. 文档

    * [JSONPath -【重要】](https://kubernetes.io/docs/reference/kubectl/jsonpath/) - 处理JSON数据接口
    * [过滤K8S输出的内容](List All Container Images Running in a Cluster)
    * Resources
        * [Managing Compute Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) - 
        分配容器资源（CPU、内存等）
        * [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) - 
    根据`namespace`分配资源
    * [Configuring kubelet Garbage Collection - 【重要】](https://kubernetes.io/docs/concepts/cluster-administration/kubelet-garbage-collection/) - 
    回收镜像/容器资源
    * [Troubleshooting your application](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/) - 
    Useful for users who are deploying code into Kubernetes and wondering why it is not working.
    * [Troubleshooting your cluster](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/) - 
    Useful for cluster administrators and people whose Kubernetes cluster is unhappy.
    * [Creating a Custom Cluster from Scratch](https://kubernetes.io/docs/getting-started-guides/scratch/)
    * [Building Large Clusters](https://kubernetes.io/docs/admin/cluster-large/)
    * [Running in Multiple Zones](https://kubernetes.io/docs/admin/multiple-zones/)
    * [Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/)
    * [API Conventions](https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md)
    * [Community Contributors(各种规范文档)](https://github.com/kubernetes/community/tree/master/contributors)
    * [Accessing Clusters - 【重要】](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#python-client)
    * [修改/etc/hosts - 【重要】](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)
    * [Pod层面配置DNS - 【重要】](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
    * [Auditing - 提供`Log backend`/`Webhook backend`监控事件](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)
    * [调试Services及网络问题](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
    * [Pod运行失败及定制错误信息](https://kubernetes.io/docs/tasks/debug-application-cluster/determine-reason-pod-failure/)
    * [`Horizontal Pod Autoscaler` 扩容/缩容 算法](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/autoscaling/horizontal-pod-autoscaler.md#autoscaling-algorithm)











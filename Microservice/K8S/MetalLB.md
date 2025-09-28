# MetalLB

> 注：下文为Deepseek生成

MetalLB 是一个专为裸金属（Bare Metal）Kubernetes 集群设计的负载均衡器实现，它通过标准路由协议（ARP/NDP 或 BGP）为 `LoadBalancer` 类型的 Service 分配外部可访问的 IP 地址。以下是其功能、优势、安装及使用指南：

---

### **一、功能与优势**
#### **功能**
1. **IP 地址分配**
   - 为 `LoadBalancer` 类型的 Service 自动分配外部 IP 地址（需预先配置 IP 地址池）。
2. **支持两种模式**
   - **Layer 2 模式**：使用 ARP（IPv4）或 NDP（IPv6）广播，将流量导向单个节点，再通过 kube-proxy 转发到 Pod。
   - **BGP 模式**：与外部路由器通过 BGP 协议交换路由信息，实现真正的负载均衡（多节点流量分发）。
3. **兼容性**
   - 无缝集成 Kubernetes 原生 Service API，无需修改应用配置。
4. **轻量级**
   - 无额外代理层，直接利用现有网络设施。

#### **优势**
- **解决裸金属集群的痛点**：在云厂商托管集群中，`LoadBalancer` 由云平台自动提供，而裸金属集群通常缺乏此功能，MetalLB 填补了这一空白。
- **灵活性**：支持静态 IP 地址池和动态分配，适应不同网络环境。
- **简单易用**：部署后几乎无需额外配置即可工作（Layer 2 模式）。
- **高可用性**：BGP 模式支持多节点负载均衡和故障转移。

---

### **二、安装 MetalLB**
#### **前提条件**
- Kubernetes 集群（版本 ≥ 1.13.0）。
- 集群网络插件兼容 MetalLB（Calico、Flannel、Cilium 等均支持）。
- 预留的 IP 地址范围（用于 MetalLB 分配）。

#### **安装步骤**
1. **通过 Manifest 安装**
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
   ```
   > 注：以上为 MetalLB 0.13.12 版本，请检查[最新版本](https://metallb.universe.tf/installation/)。

2. **等待组件就绪**
   ```bash
   kubectl -n metallb-system wait --for=condition=ready pod -l app=metallb
   ```

---

### **三、配置 MetalLB**
#### **1. Layer 2 模式配置**
创建 IP 地址池（示例为 `192.168.1.100-192.168.1.200`）：
```yaml
# metallb-config.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.100-192.168.1.200
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advert
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
```
应用配置：
```bash
kubectl apply -f metallb-config.yaml
```

#### **2. BGP 模式配置（需路由器支持）**
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: bgp-pool
  namespace: metallb-system
spec:
  addresses:
  - 203.0.113.10-203.0.113.20
---
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: bgp-advert
  namespace: metallb-system
spec:
  ipAddressPools:
  - bgp-pool
  peers:
  - my-asn: 64500
    peer-asn: 64500
    peer-address: 10.0.0.1
```

---

### **四、使用示例**
创建 `LoadBalancer` 类型的 Service：
```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```
应用后，MetalLB 会自动分配 IP：
```bash
kubectl get svc nginx
```
输出示例：
```
NAME    TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
nginx   LoadBalancer   10.96.0.100   192.168.1.100   80:30000/TCP   1m
```

---

### **五、常见问题**
1. **IP 分配失败**
   - 检查 IP 地址池是否被占用或超出子网范围。
2. **节点间流量不均（Layer 2 模式）**
   - 切换到 BGP 模式或确保所有节点均参与流量转发。
3. **BGP 会话未建立**
   - 验证路由器配置（ASN、IP 和密码匹配）。

---

### **六、卸载 MetalLB**
```bash
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```

---

通过 MetalLB，裸金属集群可以获得与云平台类似的负载均衡体验，适合本地数据中心或边缘计算场景。根据网络环境选择合适的模式（推荐从 Layer 2 开始测试）。
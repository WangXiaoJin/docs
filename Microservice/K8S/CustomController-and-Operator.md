# CustomController/Operator

> 注：此文档通过AI生成`
`
## CustomController vs Operator

以下是 Kubernetes 中 **Custom Controller（自定义控制器）** 和 **Operator** 的核心区别及实现对比的详细分析：

### 一、核心区别

| **维度**         | **Custom Controller**                          | **Operator**                                  |
|------------------|------------------------------------------------|-----------------------------------------------|
| **设计目标**     | 通用资源自动化（如自动扩缩、事件响应）         | 管理有状态应用的全生命周期（部署、备份、升级等） |
| **组成部分**     | 控制器逻辑（无需 CRD）                         | 控制器逻辑 + CRD（必须） + 领域知识编码        |
| **复杂度**       | 较低（仅处理资源事件）                         | 较高（需处理状态持久化、错误恢复、复杂运维逻辑） |
| **适用场景**     | 简单自动化任务（如根据标签触发操作）           | 数据库、消息队列、分布式系统等有状态应用        |
| **实现工具**     | 直接基于 `client-go` 开发                       | 使用 Operator SDK、Kubebuilder 等框架          |
| **资源依赖**     | 可操作现有资源（如 Pod、Deployment）            | 必须定义 CRD（如 `RedisCluster`）               |
| **状态管理**     | 通常无状态（仅响应事件）                       | 需管理应用状态（如记录当前版本、健康状态）      |

### 二、实现对比

#### **1. 架构差异**
- **Custom Controller**：
   - 基于 `Informer/Workqueue` 监听资源变化，触发事件处理逻辑。
   - 通常直接操作 Kubernetes 内置资源（如 Pod、Service）。
  ```go
  // 示例：监听 Pod 并打印事件
  informer.AddEventHandler(cache.ResourceEventHandlerFuncs{
      AddFunc: func(obj interface{}) {
          pod := obj.(*corev1.Pod)
          fmt.Println("Pod added:", pod.Name)
      },
  })
  ```

- **Operator**：
   - 基于 **Reconcile Loop**（协调循环），持续比较期望状态与实际状态。
   - 依赖 CRD 定义应用专属资源（如 `MySQLCluster`），并操作关联资源（如 StatefulSet、ConfigMap）。
  ```go
  func (r *NginxAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
      // 获取 CR 实例
      nginxApp := &operatorv1.NginxApp{}
      if err := r.Get(ctx, req.NamespacedName, nginxApp); err != nil {
          return ctrl.Result{}, err
      }
      // 创建或更新关联资源
      return ctrl.Result{}, r.createOrUpdateDeployment(nginxApp)
  }
  ```

#### **2. 代码生成工具**
- **Custom Controller**：
   - 手动编写资源监听和操作逻辑，使用动态客户端（`dynamic.Client`）或强类型客户端。
  ```go
  // 动态客户端操作示例
  podUnstructured := unstructuredFromPod(pod)
  _, err := dynamicClient.Resource(podGVR).Namespace(namespace).Create(ctx, podUnstructured, metav1.CreateOptions{})
  ```

- **Operator**：
   - 使用 **Kubebuilder** 或 **Operator SDK** 自动生成代码框架（包括 CRD、Reconciler 等）。
  ```bash
  # 使用 Kubebuilder 初始化 Operator
  kubebuilder init --domain my.domain
  kubebuilder create api --group webapp --version v1 --kind Guestbook
  ```

#### **3. 状态管理**
- **Custom Controller**：
   - 通常不维护复杂状态，仅响应事件执行动作。
  ```go
  // 简单事件处理：创建 Pod 后发送通知
  AddFunc: func(obj interface{}) {
      pod := obj.(*corev1.Pod)
      sendNotification("Pod created: " + pod.Name)
  }
  ```

- **Operator**：
   - 在 CR 的 `status` 字段中记录应用状态，实现状态机逻辑。
  ```go
  // 更新 CR 状态
  nginxApp.Status.ReadyReplicas = deployment.Status.ReadyReplicas
  if err := r.Status().Update(ctx, nginxApp); err != nil {
      return ctrl.Result{}, err
  }
  ```

#### **4. 错误处理与重试**
- **Custom Controller**：
   - 需手动实现重试逻辑（如使用工作队列的重试机制）。
  ```go
  queue := workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())
  queue.AddRateLimited(key) // 失败时重新入队
  ```

- **Operator**：
   - 框架自动处理重试（通过 `Reconcile` 返回值控制重试间隔）。
  ```go
  return ctrl.Result{RequeueAfter: 5 * time.Minute}, nil // 5 分钟后重试
  ```

### 三、选择建议

| **场景**                     | **推荐方案**          |
|------------------------------|-----------------------|
| 简单资源自动化（如清理资源） | Custom Controller     |
| 无状态应用的自动化运维       | Custom Controller     |
| 有状态应用的复杂运维         | Operator              |
| 需要定义领域专属 API         | Operator（必须 CRD） |

### 四、总结

- **Custom Controller** 是 Kubernetes 扩展的基础，适合处理通用资源操作。
- **Operator** 是 Custom Controller 的进阶形态，通过 **CRD + 领域逻辑编码** 实现复杂应用的全生命周期管理。
- 实现上：
   - Custom Controller 更偏向“事件驱动”，代码自由度更高。
   - Operator 更偏向“状态驱动”，依赖框架提供的协调机制。

选择时需根据场景复杂度权衡：**简单任务用 Controller，复杂运维用 Operator**。

---

## Custom Controller

以下是关于 Kubernetes **自定义控制器（Custom Controller）** 的作用解析和一个极简的 Demo 实现：

### **一、自定义控制器的作用**
自定义控制器是 Kubernetes 的核心扩展机制之一，用于实现以下目标：
1. **自动化运维**：监控集群中资源的状态（如 Pod、Deployment 或自定义资源），并根据规则自动修复或调整。
2. **扩展 API**：通过结合 **CRD（Custom Resource Definition）** 定义新的资源类型（如 `MyApp`），并为这些资源提供逻辑实现。
3. **领域逻辑封装**：将特定业务逻辑（如自动部署中间件、处理有状态服务）编码到控制器中，减少人工操作。

### **二、极简 Demo：自动创建 Pod 的控制器**
#### **场景描述**
1. 定义 CRD `PodCreator`，当用户创建一个 `PodCreator` 实例时，控制器自动创建一个 Pod。
2. 当 `PodCreator` 被删除时，自动清理对应的 Pod。

#### **环境准备**
1. 安装 Kubernetes 集群（如 Minikube）。
2. 安装 `kubectl` 和 Go 环境（1.16+）。
3. 安装代码生成工具（可选，用于快速生成 CRD 代码）：
   ```bash
   go install k8s.io/code-generator/cmd/client-gen@latest
   ```

#### **步骤 1：定义 CRD**
创建文件 `crd.yaml`：
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: podcreators.demo.k8s.io
spec:
  group: demo.k8s.io
  names:
    kind: PodCreator
    plural: podcreators
    singular: podcreator
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                image:
                  type: string
                command:
                  type: array
                  items:
                    type: string
```

部署 CRD：
```bash
kubectl apply -f crd.yaml
```

#### **步骤 2：编写控制器代码**
创建 Go 项目并安装依赖：
```bash
go mod init pod-creator-controller
go get k8s.io/client-go@v0.26.0
go get k8s.io/apimachinery@v0.26.0
```

创建 `main.go`：
```go
package main

import (
	"context"
	"fmt"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime/schema"
	"k8s.io/client-go/dynamic"
	"k8s.io/client-go/dynamic/dynamicinformer"
	"k8s.io/client-go/tools/cache"
	"k8s.io/client-go/tools/clientcmd"
	"time"
)

func main() {
	// 1. 加载 kubeconfig 配置
	config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
	if err != nil {
		panic(err)
	}

	// 2. 创建动态客户端
	dynamicClient, err := dynamic.NewForConfig(config)
	if err != nil {
		panic(err)
	}

	// 3. 定义监听的资源类型（PodCreator）
	gvr := schema.GroupVersionResource{
		Group:    "demo.k8s.io",
		Version:  "v1",
		Resource: "podcreators",
	}

	// 4. 创建 Informer 监听资源变化
	factory := dynamicinformer.NewFilteredDynamicSharedInformerFactory(dynamicClient, 10*time.Minute, corev1.NamespaceAll, nil)
	informer := factory.ForResource(gvr).Informer()

	// 5. 注册事件处理函数
	informer.AddEventHandler(cache.ResourceEventHandlerFuncs{
		AddFunc: func(obj interface{}) {
			// 当创建 PodCreator 时，创建 Pod
			unstructuredObj := obj.(*unstructured.Unstructured)
			name := unstructuredObj.GetName()
			namespace := unstructuredObj.GetNamespace()
			spec := unstructuredObj.Object["spec"].(map[string]interface{})

			pod := &corev1.Pod{
				ObjectMeta: metav1.ObjectMeta{
					Name:      name + "-pod",
					Namespace: namespace,
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{
						{
							Name:    "main",
							Image:   spec["image"].(string),
							Command: toStringSlice(spec["command"].([]interface{})),
						},
					},
					RestartPolicy: corev1.RestartPolicyNever,
				},
			}

			_, err := dynamicClient.Resource(corev1.SchemeGroupVersion.WithResource("pods")).
				Namespace(namespace).
				Create(context.Background(), unstructuredFromPod(pod), metav1.CreateOptions{})
			if err != nil {
				fmt.Printf("Failed to create Pod: %v\n", err)
			} else {
				fmt.Printf("Created Pod: %s\n", pod.Name)
			}
		},
		DeleteFunc: func(obj interface{}) {
			// 当删除 PodCreator 时，删除对应的 Pod
			unstructuredObj := obj.(*unstructured.Unstructured)
			podName := unstructuredObj.GetName() + "-pod"
			err := dynamicClient.Resource(corev1.SchemeGroupVersion.WithResource("pods")).
				Namespace(unstructuredObj.GetNamespace()).
				Delete(context.Background(), podName, metav1.DeleteOptions{})
			if err != nil {
				fmt.Printf("Failed to delete Pod: %v\n", err)
			} else {
				fmt.Printf("Deleted Pod: %s\n", podName)
			}
		},
	})

	// 6. 启动控制器
	stopCh := make(chan struct{})
	defer close(stopCh)
	factory.Start(stopCh)
	<-stopCh
}

// 辅助函数：转换 interface{} 切片为字符串切片
func toStringSlice(s []interface{}) []string {
	result := make([]string, len(s))
	for i, v := range s {
		result[i] = v.(string)
	}
	return result
}

// 辅助函数：将 Pod 转换为 Unstructured 对象
func unstructuredFromPod(pod *corev1.Pod) *unstructured.Unstructured {
	obj, _ := runtime.DefaultUnstructuredConverter.ToUnstructured(pod)
	return &unstructured.Unstructured{Object: obj}
}
```

#### **步骤 3：运行控制器**
```bash
go run main.go
```

#### **步骤 4：测试**
1. 创建 `PodCreator` 实例：
   ```yaml
   apiVersion: demo.k8s.io/v1
   kind: PodCreator
   metadata:
     name: my-test
   spec:
     image: busybox
     command: ["echo", "Hello from PodCreator!"]
   ```
   ```bash
   kubectl apply -f podcreator.yaml
   ```
2. 检查 Pod 是否被创建：
   ```bash
   kubectl get pods
   ```
3. 删除 `PodCreator` 实例，观察 Pod 是否被删除。

### **三、核心逻辑解析**
1. **Informer 机制**：监听 Kubernetes API 中 `PodCreator` 资源的变化。
2. **事件处理**：
   - **AddFunc**：当创建 `PodCreator` 时，根据其配置自动创建 Pod。
   - **DeleteFunc**：当删除 `PodCreator` 时，自动清理关联的 Pod。
3. **动态客户端（Dynamic Client）**：用于操作任意类型的 Kubernetes 资源（包括自定义资源）。

### **四、进阶方向**
1. **代码生成**：使用 `client-gen` 生成强类型客户端代码，替代动态客户端。
2. **状态管理**：在 `PodCreator` 的 `status` 字段中记录 Pod 的状态。
3. **错误处理**：添加重试机制和事件上报。
4. **部署控制器**：将控制器打包为容器，通过 Deployment 运行在集群内。

通过这个 Demo，可以直观理解自定义控制器如何通过监听资源变化实现自动化操作。它是构建 Operator 的基础，也是扩展 Kubernetes 能力的核心手段。

---

## Operator

以下是关于 Kubernetes **Operator** 的作用解析和一个极简的 Demo 实现：

### 一、Operator 的核心作用
Operator 是 Kubernetes 的 **自动化运维高级模式**，核心目标是为 **有状态应用**（如数据库、中间件）提供全生命周期管理。其核心能力包括：

| **能力**               | **说明**                                                                 |
|------------------------|-------------------------------------------------------------------------|
| **声明式 API**         | 通过 CRD（Custom Resource Definition）定义应用专属资源（如 `MySQLCluster`） |
| **自动化运维**         | 自动处理部署、扩缩容、备份、升级等运维操作                                |
| **状态感知与自愈**     | 持续监控应用状态，自动修复异常（如节点故障）                              |
| **领域知识编码**       | 将运维经验转化为代码（如“如何安全升级 PostgreSQL 集群”）                   |


### 二、极简 Demo：Nginx 版本管理 Operator
#### **场景描述**
1. 定义 CRD `NginxApp`，用户通过创建 `NginxApp` 资源声明期望的 Nginx 版本和副本数。
2. Operator 自动创建对应的 Deployment 和 Service。
3. 当用户修改 `NginxApp` 的 `version` 或 `replicas` 时，Operator 自动更新 Deployment。

#### **环境准备**
1. Kubernetes 集群（Minikube 或 Kind）
2. 安装 `kubectl` 和 Go 1.16+
3. 安装 Operator 开发框架（可选，但推荐使用 Kubebuilder）：
   ```bash
   go install sigs.k8s.io/kubebuilder/v3/cmd/kubebuilder@latest
   ```

### 三、实现步骤

#### **步骤 1：定义 CRD**
创建文件 `crd.yaml`：
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: nginxapps.operator.demo
spec:
  group: operator.demo
  names:
    kind: NginxApp
    plural: nginxapps
    singular: nginxapp
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                version:
                  type: string  # Nginx 版本（如 "1.25"）
                replicas:
                  type: integer  # 副本数
```

部署 CRD：
```bash
kubectl apply -f crd.yaml
```

#### **步骤 2：编写 Operator 核心逻辑**
创建 `main.go`（使用 `client-go` 和 `controller-runtime` 库）：
```go
package main

import (
	"context"
	"fmt"
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	operatorv1 "operator-demo/api/v1"
)

// NginxApp Reconciler（核心协调逻辑）
type NginxAppReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

func (r *NginxAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	// 1. 获取 NginxApp 实例
	var nginxApp operatorv1.NginxApp
	if err := r.Get(ctx, req.NamespacedName, &nginxApp); err != nil {
		return ctrl.Result{}, client.IgnoreNotFound(err)
	}

	// 2. 创建或更新 Deployment
	deployment := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      nginxApp.Name + "-deploy",
			Namespace: nginxApp.Namespace,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &nginxApp.Spec.Replicas,
			Selector: &metav1.LabelSelector{
				MatchLabels: map[string]string{"app": nginxApp.Name},
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: map[string]string{"app": nginxApp.Name},
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{{
						Name:  "nginx",
						Image: "nginx:" + nginxApp.Spec.Version,
						Ports: []corev1.ContainerPort{{ContainerPort: 80}},
					}},
				},
			},
		},
	}

	// 3. 应用 Deployment（存在则更新，不存在则创建）
	if err := r.Patch(ctx, deployment, client.Apply, client.FieldOwner("nginx-operator")); err != nil {
		return ctrl.Result{}, fmt.Errorf("failed to apply Deployment: %v", err)
	}

	return ctrl.Result{}, nil
}

func main() {
	// 初始化 Controller Manager
	mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
		Scheme:             runtime.NewScheme(),
		MetricsBindAddress: "0", // 禁用监控端口（仅演示）
	})
	if err != nil {
		panic(err)
	}

	// 注册 Reconciler
	if err = (&NginxAppReconciler{
		Client: mgr.GetClient(),
		Scheme: mgr.GetScheme(),
	}).SetupWithManager(mgr); err != nil {
		panic(err)
	}

	// 启动 Operator
	if err := mgr.Start(ctrl.SetupSignalHandler()); err != nil {
		panic(err)
	}
}
```

#### **步骤 3：运行 Operator**
```bash
go mod init operator-demo
go mod tidy
go run main.go
```

#### **步骤 4：测试**
1. 创建 `NginxApp` 实例：
   ```yaml
   apiVersion: operator.demo/v1
   kind: NginxApp
   metadata:
     name: my-nginx
   spec:
     version: "1.25"  # 指定 Nginx 版本
     replicas: 2      # 副本数
   ```
   ```bash
   kubectl apply -f nginxapp.yaml
   ```
2. 检查 Deployment 和 Pod：
   ```bash
   kubectl get deployments,po
   ```
3. 修改 `nginxapp.yaml` 中的 `version` 或 `replicas`，重新应用：
   ```bash
   kubectl apply -f nginxapp.yaml
   ```
4. 观察 Deployment 是否自动更新。

### 四、Operator vs 自定义控制器
| **特性**           | **自定义控制器**                | **Operator**                          |
|--------------------|--------------------------------|---------------------------------------|
| **核心目标**       | 通用资源操作                   | 管理有状态应用的全生命周期            |
| **复杂度**         | 较低                          | 较高（需处理状态持久化、备份等逻辑）  |
| **典型依赖**       | `client-go`                   | Operator SDK/Kubebuilder + CRD        |
| **适用场景**       | 简单自动化任务                 | 数据库、消息队列、分布式系统          |

### 五、进阶方向
1. **状态管理**：在 CR 的 `status` 字段中记录应用状态（如当前版本、健康状态）。
2. **事件处理**：添加 Kubernetes Event 记录操作日志。
3. **Webhook**：实现校验逻辑（如禁止降级版本）。
4. **打包部署**：将 Operator 打包为 Helm Chart 或 OLM Bundle。

通过这个 Demo，可以看到 Operator 如何通过 CRD + 协调循环（Reconcile Loop）实现应用的全生命周期管理。它是 Kubernetes 生态中**将运维知识代码化**的最佳实践。

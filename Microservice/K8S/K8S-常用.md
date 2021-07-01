# K8S-常用

## 组件

### Container-Management-Platform

#### KubeSphere

KubeSphere 是在 Kubernetes 之上构建的面向云原生应用的 容器混合云，支持多云与多集群管理，提供全栈的 IT 自动化运维的能力，
简化企业的 DevOps 工作流。KubeSphere 提供了运维友好的向导式操作界面，帮助企业快速构建一个强大和功能丰富的容器云平台。
KubeSphere 愿景是打造一个基于 Kubernetes 的云原生分布式操作系统，它的架构可以很方便地与云原生生态进行即插即用（plug-and-play）的集成。

> [KubeSphere](https://github.com/kubesphere/kubesphere)

#### Rancher

ancher is an open source container management platform built for organizations that deploy containers in production.
Rancher makes it easy to run Kubernetes everywhere, meet IT requirements, and empower DevOps teams.

> [Rancher](https://github.com/rancher/rancher)


### awesome-operators

A resource tracking a number of Operators out in the wild.

Operators are Kubernetes native applications. We define native as being both managed using the Kubernetes APIs
via kubectl and ran on Kubernetes as containers. Operators take advantage of Kubernetes’s extensibility to deliver the
automation advantages of cloud services like provisioning, scaling, and backup/restore while being able to run anywhere
that Kubernetes can run.

> [awesome-operators](https://github.com/operator-framework/awesome-operators)


### `Lifecycle`/`Auditing`/`Hook`/`Event`

* Admission Controllers
    * [Using Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
    * [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
    * [Admission control plugin: EventRateLimit - 原理](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/admission_control_event_rate_limit.md)
* [Auditing](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)
* Lifecycle
    * [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
    * [Container Lifecycle Hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)
* [Using Watch with the Kubernetes API](https://www.baeldung.com/java-kubernetes-watch)

* 开源组件
    * [Argo Events](https://github.com/argoproj/argo-events/)
    * [kubewatch](https://github.com/bitnami-labs/kubewatch)
    * [Eventrouter](https://github.com/heptiolabs/eventrouter)
    * [kube-eventer](https://github.com/AliyunContainerService/kube-eventer)
    * [kubernetes-event-exporter](https://github.com/opsgenie/kubernetes-event-exporter)


## 问题

### setting cgroup config for procHooks process caused

**OCI runtime create failed: container_linux.go:348: starting container process caused "process_linux.go:402:
container init caused \"process_linux.go:367: setting cgroup config for procHooks process caused \\\"failed to
write 200000 to cpu.cfs_quota_us: write /sys/fs/cgroup/cpu,cpuacct/container.slice/kubepods/burstable/pod6ba8075b-132e-11e9-ab2e-246e9674888c/a5f752a5a36fafeab7f16beb4763521cf2370efc3ba961e85a8ac1faef721b48/cpu.cfs_quota_us:
invalid argument\\\"\"": unknown**

可通过升级内核解决: <https://github.com/kubernetes/kubernetes/issues/72878>

## 文档

* [Write a simple Kubernetes Operator in Java using the Fabric8 Kubernetes Client](https://developers.redhat.com/blog/2019/10/07/write-a-simple-kubernetes-operator-in-java-using-the-fabric8-kubernetes-client/) -


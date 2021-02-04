# K8S-常用

## 组件

* [awesome-operators](https://github.com/operator-framework/awesome-operators) - A resource tracking a number of Operators out in the wild.

## 问题

### setting cgroup config for procHooks process caused

**OCI runtime create failed: container_linux.go:348: starting container process caused "process_linux.go:402:
container init caused \"process_linux.go:367: setting cgroup config for procHooks process caused \\\"failed to
write 200000 to cpu.cfs_quota_us: write /sys/fs/cgroup/cpu,cpuacct/container.slice/kubepods/burstable/pod6ba8075b-132e-11e9-ab2e-246e9674888c/a5f752a5a36fafeab7f16beb4763521cf2370efc3ba961e85a8ac1faef721b48/cpu.cfs_quota_us:
invalid argument\\\"\"": unknown**

可通过升级内核解决: <https://github.com/kubernetes/kubernetes/issues/72878>




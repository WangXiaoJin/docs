# Helm

## 参考文档

* [官方文档](https://helm.sh/zh/docs/)
  * [Helm和Kubernetes版本兼容](https://helm.sh/zh/docs/topics/version_skew/)
  * [chart模板配置技巧](https://helm.sh/zh/docs/howto/charts_tips_and_tricks/)
  * [利用Github自动发布Chart](https://helm.sh/zh/docs/howto/chart_releaser_action/)
  * [Chart的目录结构及详解](https://helm.sh/zh/docs/topics/charts/)

* [Go模板](https://pkg.go.dev/text/template)


## 安装

我当前的系统是`Linux x86_64`，`Kubernetes 1.18.8`，对应下载`Helm 3.6.3`。

```shell
shell> wget https://get.helm.sh/helm-v3.6.3-linux-amd64.tar.gz
shell> tar -zxvf helm-v3.6.3-linux-amd64.tar.gz
shell> mv linux-amd64/helm /usr/local/bin/helm
shell> helm version
# 命令自动补全
shell> helm completion bash | sudo tee /etc/bash_completion.d/helm
# 添加一个常用仓库
shell> helm repo add bitnami https://charts.bitnami.com/bitnami
```


* [Helm常用功能介绍](https://helm.sh/zh/docs/intro/using_helm/)
  * `helm search hub`
  * `helm search repo`
  * `helm install`、`helm install -f values.yaml bitnami/wordpress --generate-name`、`--set 参数格式`
  * `helm status`
  * `helm show values`
  * `helm get values <release-name>`
  * `helm upgrade`
  * `helm rollback [RELEASE] [REVISION]`
  * `helm uninstall [RELEASE]`
  * `helm list`
  * `helm repo`
  * `helm lint`
  * `helm package`

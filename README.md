## 相关简介

k8s的安装选择版本为1.18.0，因为有的插件存在较大的兼容问题，一定要看好版本

### Flannel

k8s支持的一种网络模式，新手刚刚接触k8s推荐使用它，安装简单，但是确实不太稳定，而且性能也不是太好，在生产环境中并不是很推荐

### kubernetes Dashboard （版本2.0.0）

Kubernetes Dashboard 是 Kubernetes 的官方 Web UI。使用 Kubernetes Dashboard，你可以：

- 向 Kubernetes 集群部署容器化应用
- 诊断容器化应用的问题
- 管理集群的资源
- 查看集群上所运行的应用程序
- 创建、修改Kubernetes 上的资源（例如 Deployment、Job、DaemonSet等）
- 展示集群上发生的错误

例如：您可以伸缩一个 Deployment、执行滚动更新、重启一个 Pod 或部署一个新的应用程序


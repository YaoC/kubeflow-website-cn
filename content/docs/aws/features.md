+++
title = "AWS 上 Kubeflow 的功能"
weight = 200
+++

## 使用 AWS 上的 Kubeflow 的原因

在亚马逊 EKS 上运行 Kubeflow 可以带来如下可选和可配置的功能：

* 你可以使用 **[eksctl](https://github.com/weaveworks/eksctl)** 来管理你的亚马逊 EKS 集群，并且可以很容易的在多个计算节点和 GPU 工作节点之间进行选择。
* 你可以使用 **[AWS ALB Ingress 控制器](https://github.com/kubernetes-sigs/aws-alb-ingress-controller)** 来管理 ingress 流量。
* 你可以利用 **[Amazon FSx CSI 驱动](https://github.com/kubernetes-sigs/aws-fsx-csi-driver)** 来管理专门为计算密集型工作负载（比如高性能计算和机器学习）优化过的 Lustre 文件系统。亚马逊 FSx 能扩展到数百 GBps 的吞吐量和百万级的 IOPS。
* 将 Kubernetes 集群日志集中并统一放在 **[Amazon CloudWatch](https://aws.amazon.com/cloudwatch/)**，它可以帮助排查和解决故障。
* 你可以使用 **[AWS Certificate Manager](https://aws.amazon.com/certificate-manager/)** 和 **[AWS Cognito](https://aws.amazon.com/cognito/)** 开启 TLS 和用户认证。
* 你可以为你的 Kubernetes 集群的 API 服务端点开启 **[私密访问](https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html)**。
* 你在 AWS 上部署的 Kubeflow 可以自动检测到 GPU 工作节点并且安装 **[NVIDIA 设备插件](https://github.com/NVIDIA/k8s-device-plugin)**。

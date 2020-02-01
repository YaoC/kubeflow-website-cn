+++
title = "在已存在的集群中初始化设置"
description = "配置一个集群如果你已经有了一个"
weight = 5
+++

## 为已存在的集群初始化设置

获取 Kubeconfig 文件：

	az aks get-credentials -n <NAME> -g <RESOURCE_GROUP_NAME>

从这里开始，请参考 [安装 Kubeflow](/docs/azure/deploy/install-kubeflow)。
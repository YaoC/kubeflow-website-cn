+++
title = "私密访问"
description = "如何创建私有 EKS 集群"
weight = 80
+++

本章节帮助你在你的亚马逊 EKS 集群的 Kubernetes API 服务端点开启私密访问，并且完全禁止公开访问以便它不能被公网访问到。

## 为你的集群 API 服务端点开启私密访问

你可以给 Kubernetes API 服务器开启私密访问，这样在你的工作节点和 API 服务器之间的所有通讯都会限定在你的 VPC 中。你也可以完全禁用公网访问你的 API 服务器以便它不能被公网访问到。

你可以在 `${KF_DIR}/aws_config/cluster_features.sh` 中开启私密访问：

```shell
PRIVATE_LINK=false
ENDPOINT_PUBLIC_ACCESS=true
ENDPOINT_PRIVATE_ACCESS=false
```

默认情况下，API 服务端点是对互联网开放的 (`ENDPOINT_PUBLIC_ACCESS=true`)，同时访问 API 服务器是安全的，它使用了集成的 [AWS 身份访问管理系统 (IAM)](https://aws.amazon.com/iam/) 和原生的 Kubernetes [基于角色的访问控制系统](https://kubernetes.io/docs/admin/authorization/rbac/) (`ENDPOINT_PRIVATE_ACCESS=false`)。

你可以给 Kubernetes API 服务器开启私密访问，这样在你的工作节点和 API 服务器之间的所有通讯都会限定在你的 VPC 中 (`ENDPOINT_PRIVATE_ACCESS=true`)。你也可以完全禁用公网访问你的 API 服务器以便它不能被公网访问到 (`ENDPOINT_PUBLIC_ACCESS=false`)。在这种情况下，你需要在你的 VPC 中有一个实例去跟你的 Kubernetes API 服务器通讯。

注意：如果你集成的方式不正确的话你可能会看到 `InvalidParameterException` 错误。

请查看 [Amazon EKS 集群端点访问控制](https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html) 以获取更多细节。

+++
title = "日志"
description = "为 Kubeflow 添加日志支持"
weight = 70
+++

亚马逊 EKS 控制平面日志系统直接提供审计和诊断日志，这些日志来自于从亚马逊 EKS 控制平面到 [CloudWatch](https://aws.amazon.com/cloudwatch/) 记录的你的账户的日志。
这些日志让你安全运行你的集群变得容易。你可以精确地选择你想要的日志类型，日志就会被作为日志流发送到一个包含每个在 [CloudWatch](https://aws.amazon.com/cloudwatch/) 中的亚马逊 EKS 集群组中。

如果你看一下 `${KF_DIR}/aws_config/cluster_features.yaml`，你可以看到以下配置：

```shell
CONTROL_PLANE_LOGGING=false
CONTROL_PLANE_LOGGING_COMPONENTS=api,audit,authenticator,controllerManager,scheduler

WORKER_NODE_GROUP_LOGGING=false
```

默认情况下，集群控制平面日志和工作节点组日志不会被发送到 CloudWatch 日志中。你必须逐个开启每个日志类型从而能为你的集群发送日志。

### 控制平面日志

你可以更新 `CONTROL_PLANE_LOGGING=true` 来开启控制平面日志以及自定义你想要收集日志的组件。只有以下这些组件可用，你只能在这些组件中使用命令。

* api
* audit
* authenticator
* controllerManager
* scheduler

打开 [CloudWatch 控制台](https://console.aws.amazon.com/cloudwatch/home#logs:prefix=/aws/eks)，选择你想要查看日志的集群。日志组名称格式为 `/aws/eks/${AWS_CLUSTER_NAME}/cluster`。

> 注意：如果你设置了 `CONTROL_PLANE_LOGGING=false`，`CONTROL_PLANE_LOGGING_COMPONENTS` 的值将不会被使用。

### 工作节点组日志

你可以更新 `WORKER_NODE_GROUP_LOGGING=true` 来开启工作节点组日志，所有的 pod 日志都会被发送到 CloudWatch。日志组名称格式为 `/eks/${AWS_CLUSTER_NAME}/containers`。

如果在你已经创建集群后想要改变日志设置，请查看 [这里](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-logs.html) 了解细节。

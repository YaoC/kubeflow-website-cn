+++
title = "在 AWS 上定制化 Kubeflow"
description = "调整 Kubeflow 的 AWS 部署方式"
weight = 20
+++

本向导描述了如何定制化你在亚马逊 EKS 上的 Kubeflow 部署方式。
这些步骤在你运行 `apply -V -f ${CONFIG_FILE}` 命令之前可以结束。请看下面的段落了解详情。如果你不理解部署过程，请查看 [部署](/docs/aws/deploy) 以了解细节。

## 定制化 Kubeflow

这是 AWS 平台上 kfctl 可选的配置参数：

| 选项  | 描述  | 是否必需 |
|---|---|---|
| `awsClusterName` | 你的新的或已存在的亚马逊 EKS 集群的名称 | 是 |
| `awsRegion`  | AWS 集群的启动地区 |  是 |
| `awsNodegroupRoleNames`  | 你的工作节点的 IAM 角色名称 | 已存在的集群必需 / 新集群可选 |


### Customize your Amazon EKS cluster

在你运行 `kfctl apply -V -f ${CONFIG_FILE}` 之前，你可以在你创建集群之前编辑集群配置文件来改变集群规格。

集群配置保存在 `${KF_DIR}/aws_config/cluster_config.yaml`。请查看 [eksctl](https://eksctl.io/) 以了解配置细节。

举例来说，以下是一个包含单个节点组的集群的清单，该节点组含有 2 个 `p2.xlarge` 实例。你可以很容易地开启 SSH 并配置一个公钥。所有的工作节点都会在单个可用区中。

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  # AWS_CLUSTER_NAME and AWS_REGION will override `name` and `region` here.
  name: kubeflow-example
  region: us-west-2
  version: '1.13'
# If your region has multiple availability zones, you can specify 3 of them.
#availabilityZones: ["us-west-2b", "us-west-2c", "us-west-2d"]

# NodeGroup holds all configuration attributes that are specific to a nodegroup
# You can have several node groups in your cluster.
nodeGroups:
  - name: eks-gpu
    instanceType: p2.xlarge
    availabilityZones: ["us-west-2b"]
    desiredCapacity: 2
    minSize: 0
    maxSize: 2
    volumeSize: 30
    ssh:
      allow: true
      sshPublicKeyPath: '~/.ssh/id_rsa.pub'

  # Example of GPU node group
  # - name: Tesla-V100
  # Choose your Instance type for the node group.
  #   instanceType: p3.2xlarge
  # GPU cluster can use single availability zone to improve network performance
  #   availabilityZones: ["us-west-2b"]
  # Autoscaling Groups settings
  #   desiredCapacity: 0
  #   minSize: 0
  #   maxSize: 4
  # Node Root Disk
  #   volumeSize: 50
  # Enable SSH out side your VPC.
  #   allowSSH: true
  #   sshPublicKeyPath: '~/.ssh/id_rsa.pub'
  # Customize Labels
  #   labels:
  #     'k8s.amazonaws.com/accelerator': 'nvidia-tesla-k80'
  # Setup pre-defined iam roles to node group.
  #   iam:
  #     withAddonPolicies:
  #       autoScaler: true

```

### 定制化私密访问
请看 [这篇](/docs/aws/private-access)

### 定制化日志
请看 [这篇](/docs/aws/logging)

### 定制化认证
请看 [这篇](/docs/aws/authentication)

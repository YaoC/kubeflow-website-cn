+++
title = "存储选项"
description = "在 Kubeflow 中为 Lustre 使用 EFS 和 FSx"
weight = 95
+++

本向导描述如何在 Kubeflow 中为 Lustre 使用亚马逊 EFS 和 FSx。

## 亚马逊 EFS

亚马逊 EFS 在 AWS 中用来管理 NFS。亚马逊 EFS 支持 `ReadWriteMany` 访问模式，这意味着存储卷可以被挂载成被多个节点读写。
这在创建一个可以被挂载进 pod（如 Jupyter）的共享系统时很有用。举个例子，一个小组可以在整个团队中共享数据集或模型。
默认情况下，亚马逊 EFS CSI 驱动是不开启的，你需要跟随下面的步骤来安装它。

### 部署亚马逊 EFS CSI 插件

```shell
cd ${KF_DIR}/ks_app
export COMPONENT=aws-efs-csi-driver
ks generate aws-efs-csi-driver ${COMPONENT}
ks apply default -c ${COMPONENT}
```

### 静态供给

你可以去 [亚马逊 EFS 控制台](https://us-west-2.console.aws.amazon.com/efs/home) 创建一个新的亚马逊 EFS 文件系统。选择 VPC、子网 ID 以及供给模式来使用。
请特别关注安全组，并且确认到 NFS 端口 2049 的流量是否被允许。
然后你将会得到一个文件系统 ID，你就可以使用它来创建持久卷和持久卷声明了。

```shell
cd ${KF_DIR}/ks_app
export COMPONENT=efs-storage
ks generate aws-efs-pv ${COMPONENT} --efsId=${your_file_system_id}
ks apply default -c ${COMPONENT}
```

当你创建 Jupyter notebook 时使用亚马逊 EFS 作为 notebook 卷
<img src="/docs/images/aws/efs-volume.png"
  alt="Amazon EFS JupyterNotebook Volume"
  class="mt-3 mb-3 border border-info rounded">


## 亚马逊 FSx for Lustre

亚马逊 FSx for Lustre 提供一个为高速处理类型的工作负载比如机器学习和高性能计算（HPC）工作负载优化过的高性能的文件系统。
[AWS FSx for Lustre CSI 驱动](https://github.com/kubernetes-sigs/aws-fsx-csi-driver) 能够帮助 Kubernetes 用户很容易地利用这项服务。

Lustre 是支持 `ReadWriteMany` 的又一种文件系统。在 亚马逊 EFS 和 Lustre 之间的一个区别是 Lustre 可以被用来作为训练数据的缓存层，使用 S3 作为后端存储。
在你使用卷之前不需要转移数据。默认情况下，亚马逊 FSx CSI 驱动没有开启，你需要按照以下步骤来安装它。

### 部署亚马逊 FSx CSI 插件

```shell
cd ${KF_DIR}/ks_app
export COMPONENT=aws-fsx-csi-driver
ks generate aws-fsx-csi-driver ${COMPONENT}
ks apply default -c ${COMPONENT}
```

### 静态供给

你可以静态化地提供亚马逊 FSx for Lustre，然后传递文件系统 ID 和 DNS 名称来生成持久卷和持久卷声明。
它将会创建默认存储类 `fsx-default`。

```shell
cd ${KF_DIR}/ks_app
export COMPONENT=fsx-static-storage

ks generate aws-fsx-pv-static ${COMPONENT} --fsxId=fs-048xxxx7c25 --dnsName=fs-048xxxx7c25.fsx.us-west-2.amazonaws.com

ks apply default -c ${COMPONENT}
```

一旦你的持久卷声明准备好了，你可以在你的工作负载中做如下声明：

```shell
volumes:
- name: persistent-storage
  persistentVolumeClaim:
    claimName: fsx-static-storage
```


### 动态供给

你可以为你的高性能计算工作负载用以下方式动态化地提供亚马逊 FSx for Lustre 文件系统。
`SecurityGroupId` 和 `SubnetId` 是必需的。亚马逊 FSx for Lustre 是一个基于文件系统的可用区，你可以只传递一个 SubnetId。
这意味着你需要在单个可用区创建一个集群，这对机器学习工作负载是有意义的。

如果你在亚马逊 S3 已经有一个训练数据集了，作为可选项你可以传递你的 bucket 名称，它将会被亚马逊 FSx for Lustre 当做数据仓库来使用，然后带有你的训练数据集的文件系统也会被准备好。

它将会为你创建存储类、持久卷和持久卷声明等 Kubernetes 资源。

```shell
cd ${KF_DIR}/ks_app
export COMPONENT=fsx-dynamic-storage

ks generate aws-fsx-pv-dynamic ${COMPONENT} --securityGroupIds=sg-0c380xxxxxxxx --subnetId=subnet-007f9cxxxxxxx
ks param set ${COMPONENT} s3ImportPath s3://your_dataset_bucket

ks apply default -c ${COMPONENT}
```

在你的工作负载中，你只需要挂载卷并且使用 `${COMPONENT}` 作为你的持久卷声明名称。它大概会耗费 5-7 分钟的时间来准备存储。

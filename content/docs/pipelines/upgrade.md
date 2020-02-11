+++
title = "升级和重新安装"
description = "如何升级或是重新安装你的 Kubeflow Pipelines 部署"
weight = 50
+++

从 Kubeflow 0.5 版本开始，Kubeflow Pipelines 将流水线数据持久化存储到一个持久化存储卷中。Kubeflow Pipelines 因此能够支持以下功能：

* **重新安装：** 你可以删除一个集群然后创建一个新的集群，通过指定现有存储的方式让新建的集群能够检索到原来的数据。

注意 Kubeflow 目前并不支持升级。查看 [该 issue](https://github.com/kubeflow/kubeflow/issues/3727) 获取最新进展。

如果你想在 Google Cloud Platform（GCP）中使用最新的 Kubeflow Pipelines，我们建议你使用 [Kubeflow Pipelines 独立部署](/docs/pipelines/standalone-deployment-gcp/)。

## 相关环境

Kubeflow Pipelines 创建并管理以下与你的机器学习流水线相关的数据：

* **元数据：** 实验、作业、run等。Kubeflow Pipelines 将流水线元数据存储在一个 MySQL 数据库中。
* **构件：** 流水线包、指标、视图等。Kubeflow Pipelines 将构件存储在一个 [Minio 服务器](https://docs.minio.io/) 中。


 MySQL 数据库和 Minio 服务器的存储后端都是 Kubernetes
[持久化存储卷](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)（PV）子系统。 

* 如果你在 Google Cloud Platform（GCP）中部署，那么 Kubeflow Pipelines 会创建一个 Compute Engine 
  [Persistent Disk](https://cloud.google.com/persistent-disk/)（PD）并将它作为一个 PV 进行挂载。
* 如果你不是在 GCP 中部署，那么你可以指定适合自己的 PV。

## 部署 Kubeflow

本节描述一种能够确保你可以使用 Kubeflow Pipelines 重装功能的 Kubeflow 部署方法。

### 在 GCP 中部署 Kubeflow

参照指南 [在 GCP 中部署 Kubeflow](/docs/gke/deploy/)。你无需执行任何其他操作。

部署结束后，你可以在 GCP Deployment Manager 中看到两个条目，一个是部署的集群，另一个是部署的存储：

<img src="/docs/images/pipelines-deployment-storage1.png" 
  alt="Deployment Manager showing the storage deployment entry"
  class="mt-3 mb-3 border border-info rounded">

后缀为 `-storage` 的条目为元数据存储创建一个PD，为构件库创建一个PD：

<img src="/docs/images/pipelines-deployment-storage2.png" 
  alt="Deployment Manager showing details of the storage deployment entry"
  class="mt-3 mb-3 border border-info rounded">

### 在其它环境（非 GCP）中部署 Kubeflow

以下步骤假设你已经配置好一个 Kubernetes 集群。

* 如果你不需要自定义的存储同时乐意使用 Kubeflow 提供的默认 PV，你可以参照 Kubeflow
  [快速入门](/docs/started/getting-started/#kubeflow-quick-start) 部署而不用进行其它操作。部署脚本使用 Kubernetes 默认的 [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/#the-storageclass-resource) 为你提供 PV。

* 如果你想要指定一个自定义的 PV：

  1. 在你的 Kubernetes 集群中使用首选存储类型创建两个 PV。参考 [Kubernetes PV 指南](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)。

  1. 按照 Kubeflow
     [快速入门](/docs/started/getting-started/#kubeflow-quick-start)，但请注意对标准过程的以下更改：

        在执行 `apply` 命令 **之前**：

        ```
        kfctl apply -V -f ${CONFIG_FILE}
        ```

        你首先需要编辑以下文件指定你的 PV：

        `${KF_DIR}/kustomize/minio/overlays/minioPd/params.env`
        ```
        ...
        minioPd=[YOUR-PRE-CREATED-MINIO-PV-NAME]
        ...
        ```

        `${KF_DIR}/kustomize/mysql/overlays/mysqlPd/params.env`
        ```
        ...
        mysqlPd=[YOUR-PRE-CREATED-MYSQL-PV-NAME]
        ...
        ```

  1. 接着照常执行 `apply` 命令：

        ```
        kfctl apply -V -f ${CONFIG_FILE}
        ``` 

## 重新安装 Kubeflow Pipelines

你可以删除一个集群然后创建一个新的集群，通过指定现有存储的方式让新建的集群能够检索到原来的数据。

**注意：** 你必须使用命令行方式部署。你无法在 web 界面上重新安装 Kubeflow Pipelines

### 在 GCP 中重新安装 Kubeflow Pipelines

要重新安装 Kubeflow Pipelines，安装 [命令行部署说明](/docs/gke/deploy/deploy-cli/)，但是注意该过程中的以下更改：

1. 注意，当你执行 `kfctl apply` 或 `kfctl build` 命令时，你需要使用一个与你现有的 `${KF_NAME}` 名称不同的 `${KF_NAME}` 名称。
   否则你现有 PD 中的数据会在执行 `kfctl apply` 命令时被删除。

1. 在执行 `apply` 命令 **之前**：

    ```
    kfctl apply -V -f ${CONFIG_FILE}
    ```

    你首先需要：
    * 编辑 `${KF_DIR}/gcp_config/storage-kubeflow.yaml` 来跳过创建新的存储：

      ```
      ...
      createPipelinePersistentStorage: false
      ...
      ```

    * 编辑以下文件以指定在先前部署中创建的持久性磁盘：

      `${KF_DIR}/kustomize/minio/overlays/minioPd/params.env`
      ```
      ...
      minioPd=[NAME-OF-ARTIFACT-STORAGE-DISK]
      ...
      ```

      `${KF_DIR}/kustomize/mysql/overlays/mysqlPd/params.env`
      ```
      ...
      mysqlPd=[NAME-OF-METADATA-STORAGE-DISK]
      ...
      ```

1. 接着执行 `apply` 命令：

    ```
    kfctl apply -V -f ${CONFIG_FILE}
    ``` 

### 在其它环境（非GCP）中重新安装 Kubeflow 

以下步骤在非 GCP 环境中的安装都是相同的，除了必须使用与先前部署中相同的 PV 定义来在新的集群中创建 PV。

1. 在 Kubernetes 集群中使用和你之前部署相同的 PV 定义新建两个 PV。参考 [Kubernetes PV 指南](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)。

1. 按照 Kubeflow [快速入门](/docs/started/getting-started/#kubeflow-quick-start)，请注意对标准过程的以下更改：

    在执行 `apply` 命令 **之前**：

    ```
    kfctl apply -V -f ${CONFIG_FILE}
    ```

    你首先需要编辑以下文件指定你的 PV：

    `${KF_DIR}/kustomize/minio/overlays/minioPd/params.env`
    ```
    ...
    minioPd=[YOUR-PRE-CREATED-MINIO-PV-NAME]
    ...
    ```

    `${KF_DIR}/kustomize/mysql/overlays/mysqlPd/params.env`
    ```
    ...
    mysqlPd=[YOUR-PRE-CREATED-MYSQL-PV-NAME]
    ...
    ```

1. 接着执行 `apply` 命令：

    ```
    kfctl apply -V -f ${CONFIG_FILE}
    ``` 

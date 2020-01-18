+++
title = "在 GCP 中部署独立的 Pipelines"
description = "在集群中部署独立的 Pipelines 的指南"
weight = 20
+++

[Kubeflow 部署](/docs/started/getting-started/#installing-kubeflow)包括了许多组件，其中就有流水线，
除此之外你也可以只部署 Kubeflow Pipelines。
你可以按照以下指南使用我们提供的 kustomize 清单部署一个独立的 Kubeflow Pipelines。

了解 [Kubernetes](https://kubernetes.io/docs/home/)、[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) 和 [kustomize](https://kustomize.io/) 的相关知识可以帮助你更好地理解本文档的内容并根据自己的需要自定义部署。

{{% alert title="并非只能在 GCP 中安装" color="info" %}}
<p>本指南目前只介绍了如何在 Google Cloud Platform (GCP) 中部署独立的 Kubeflow Pipelines。
当然你也可以将 Kubeflow Pipelines 安装到其它平台中。本指南有待更新，参见
<a href="https://github.com/kubeflow/website/issues/1253">issue #1253</a>。</p>
{{% /alert %}}

## 共同的先决条件

以下是后续所有安装指南都需要的先决条件，只需要在首次安装时设置。

### 下载 kubectl 客户端工具
参考 [kubectl 安装指南](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。

你需要 1.14 或更高版本的 kubectl，这样才能原生支持 kustomize。

### 配置 kubectl 连接你的集群
参考 Google Kubernetes Engine (GKE) 指南来[为 kubectl 配置集群访问权限](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)。

## 在现有集群中部署独立的 Kubeflow Pipelines

1. 部署最新版的 Kubeflow Pipelines：

    ```
    export PIPELINE_VERSION={{% pipelines/latest-version %}}
    kubectl apply -k github.com/kubeflow/pipelines//manifests/kustomize/env/dev?ref=$PIPELINE_VERSION
    ```

1. 获取 Kubeflow Pipelines UI 的 URL：

    ```
    kubectl describe configmap inverse-proxy-config -n kubeflow | grep googleusercontent.com
    ```

## 从零开始部署部署独立的 Kubeflow Pipelines

1. 准备一个 Kubernetes 集群：

    参考 GKE 指南在 Google Cloud Platform (GCP) 上 [创建一个集群](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster)。

    使用以下 gcloud 命令创建一个可以运行所有流水线示例的集群：

    ```
    # 以下变量可以根据需要自定义。

    CLUSTER_NAME="kubeflow-pipelines-standalone"
    ZONE="us-central1-a"
    MACHINE_TYPE="n1-standard-2" # A machine with 2 CPUs and 7.50GB memory
    SCOPES="storage-rw,cloud-platform" # These scopes are needed for running some pipeline samples.

    gcloud container clusters create $CLUSTER_NAME \
        --zone $ZONE \
        --machine-type $MACHINE_TYPE \
        --scopes $SCOPES \
        --num-nodes 2 \
        --max-nodes 5 \
        --min-nodes 2 \
        --enable-autoscaling
    ```
     **注意**：为了方便使用，这里通过 `SCOPES="storage-rw,cloud-platform"` 让集群过度拥有了所有的 GCP 权限。
     有关更安全的集群设置方法请参考 [Pipelines 中的 GCP 权限认证](/docs/gke/authentication-pipelines/)。

    参考资料：

    - 你可以在 https://cloud.google.com/compute/docs/regions-zones/#available 查找可用区。
    - 从 https://cloud.google.com/sdk/gcloud/ 下载 gcloud 客户端工具。
    - 访问 https://cloud.google.com/sdk/gcloud/reference/container/clusters/create 
      阅读 `gcloud container clusters create` 命令的帮助文档。

1. 配置 kubectl连接你新创建的集群。参考[为 kubectl 配置集群访问权限](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)。

1. 在你的集群中独立部署最新版的 Kubeflow Pipelines：

    ```
    export PIPELINE_VERSION={{% pipelines/latest-version %}}
    kubectl apply -k github.com/kubeflow/pipelines//manifests/kustomize/env/dev?ref=$PIPELINE_VERSION
    ```

    Kubeflow Pipelines 应用需要一段时间（大约3分钟）才能启动完成。

1. 获取 Pipelines UI 的公网 URL 并用它进入 Kubeflow Pipelines：
    ```
    kubectl describe configmap inverse-proxy-config -n kubeflow | grep googleusercontent.com
    ```

## 更新
1. 配置 kubectl连接你新创建的集群。参考[为 kubectl 配置集群访问权限](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)。
1. 将 Kubeflow Pipelines 升级到你指定的版本：
    ```
    export PIPELINE_VERSION=<version-you-want-to-upgrade-to>
    kubectl apply -k github.com/kubeflow/pipelines//manifests/kustomize/env/dev?ref=$PIPELINE_VERSION
    ```
    查看 [Kubeflow Pipelines github 代码仓库](https://github.com/kubeflow/pipelines/releases)获取可用的版本。

## 自定义

通过 kustomize [叠加](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/glossary.md#overlay)完成自定义。

注意 - 以下的命令假定你已经安装了 v1.14.0 或更高版本的 kubectl，这样才能原生支持 kustomize。
访问[这里](https://kubernetes.io/docs/tasks/tools/install-kubectl/)获取 kubectl 的最新版

执行以下命令前请先克隆 [Kubeflow Pipelines 代码仓库](https://github.com/kubeflow/pipelines)并将它用作工作目录。

### 使用 CloudSQL 和 Google Cloud Storage 在 GCP 上部署

建议在生产环境使用。查看[这里](https://github.com/kubeflow/pipelines/tree/master/manifests/kustomize/env/gcp)获取更多细节。

### 更改部署的命名空间

如果要在 FOO 命名空间部署独立的 Kubeflow Pipelines：

1. 将 [dev/kustomization.yaml](https://github.com/kubeflow/pipelines/blob/master/manifests/kustomize/env/dev/kustomization.yaml)
    或 [gcp/kustomization.yaml](https://github.com/kubeflow/pipelines/blob/master/manifests/kustomize/env/gcp/kustomization.yaml) 中
 namespace 的值修改为 FOO。
1. 接着执行

```
kubectl apply -k manifests/kustomize/env/dev
# Or the following if using GCP Cloud SQL + Google Cloud Storage
# kubectl apply -k manifests/kustomize/env/gcp
```

### 禁用公网端点

默认情况下，部署会通过安装[反向代理](https://github.com/google/inverting-proxy)暴露一个公网 URL。
如果你想跳过安装，

1. 在 [kustomization.yaml](https://github.com/kubeflow/pipelines/blob/master/manifests/kustomize/base/kustomization.yaml) 中注释掉代理组件。
1. 接着执行：

```
kubectl apply -k manifests/kustomize/env/dev
# 如果是使用 GCP Cloud SQL + Google Cloud Storage 的话执行以下命令
# kubectl apply -k manifests/kustomize/env/gcp
```

UI 仍然可以通过端口转发访问：

```
kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80
```

然后打开 http://localhost:8080/。

## 卸载

你可以通过以下步骤卸载 Kubeflow Pipeline：

1. 配置 kubectl连接你新创建的集群。参考[为 kubectl 配置集群访问权限](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)。

1. 卸载 Pipelines：
    ```
    export PIPELINE_VERSION={{% pipelines/latest-version %}}
    kubectl delete -k github.com/kubeflow/pipelines//manifests/kustomize/env/dev?ref=$PIPELINE_VERSION
    ```

    如果通过 kustomize 部署则执行：

    ```
    kubectl delete -k manifests/kustomize/env/dev
    # 如果是使用 GCP Cloud SQL + Google Cloud Storage 的话执行以下命令
    # kubectl delete -k manifests/kustomize/env/gcp
    ```

## 维护自定义清单的最佳实践

### 用一个代码仓库维护你的清单

将以下内容保存到具有版本控制的代码仓库中。

文件 `kustomization.yaml`。
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
# Edit the following to change the deployment to your custom namespace.
namespace: kubeflow
# You can add other customizations here using kustomize.
# Edit ref in the following link to deploy a different version of Kubeflow Pipelines.
bases:
- github.com/kubeflow/pipelines//manifests/kustomize/env/dev?ref={{% pipelines/latest-version %}}
```

### 如何使用代码仓库来部署、更新和卸载
部署： `kubectl apply -k $YOUR_REPO`

升级：

   1. (推荐方式) 为你的 KFP 数据备份。
   1. 将 `ref={{% pipelines/latest-version %}}` 修改成你要升级的版本。

        查看 [Kubeflow Pipelines github 代码仓库](https://github.com/kubeflow/pipelines/releases)获取可用的版本。
   1. 部署： `kubectl apply -k $YOUR_REPO`.

卸载： `kubectl delete -k $YOUR_REPO`.

### 进一步阅读
* kustomize 的[推荐工作流程——复用现成的配置](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/workflows.md#off-the-shelf-configuration).

## 故障排出

### 在集群中独立安装 Kubeflow Pipelines 遇到权限错误的问题

执行：

```
kubectl create clusterrolebinding your-binding --clusterrole=cluster-admin --user=[your-user-name]
```

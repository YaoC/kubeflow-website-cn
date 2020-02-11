+++ 
title = "启用 GPU 和 TPU"
description = "在 Google Kubernetes Engine（GKE）中为 Kubeflow Pipelines 启用 GPU 和 TPU"
weight = 10
+++

本页面描述了如何使用 Pipelines DSL 语言在  GKE 中为流水线启用 GPU 或 TPU。

## 先决条件

要在你的 Kubeflow 集群中启用 GPU 和 TPU，在设置集群之前请先按照如何为
Kubeflow [自定义](/docs/gke/customizing-gke#common-customizations) GKE 集群的指南完成操作。

## 配置 ContainerOp 来使用 GPU

启用 GPU 后，Kubeflow 安装脚本会安装一个类型为 nvidia-tesla-k80 的默认 GPU 资源池，同时启用动态扩容功能。
以下代码中的 ContainerOp 使用了 2 个 GPU。

```python
import kfp.dsl as dsl
gpu_op = dsl.ContainerOp(name='gpu-op', ...).set_gpu_limit(2)
```

上面的代码会被编译成如下的 Kubernetes Pod 描述：

```yaml
container:
  ...
  resources:
    limits:
      nvidia.com/gpu: "2"
```

如果集群中有不同 GPU 类型的多个节点资源池，你可以通过以下代码指定 GPU 类型。

```python
import kfp.dsl as dsl
gpu_op = dsl.ContainerOp(name='gpu-op', ...).set_gpu_limit(2)
gpu_op.add_node_selector_constraint('cloud.google.com/gke-accelerator', 'nvidia-tesla-p4')
```

上面的代码会被编译成如下的 Kubernetes Pod 描述：


```yaml
container:
  ...
  resources:
    limits:
      nvidia.com/gpu: "2"
nodeSelector:
  cloud.google.com/gke-accelerator: nvidia-tesla-p4
```

查看 [GKE GPU 指南](https://cloud.google.com/kubernetes-engine/docs/how-to/gpus) 了解更多关于 GPU 的设置。

## 配置 ContainerOp 来使用 TPU

使用以下代码来在 GKE 中配置 ContainerOp 使用 TPU：

```python
import kfp.dsl as dsl
import kfp.gcp as gcp
tpu_op = dsl.ContainerOp(name='tpu-op', ...).apply(gcp.use_tpu(
  tpu_cores = 8, tpu_resource = 'v2', tf_version = '1.12'))
```

上面的代码使用了 8 个 v2 版本的 TPU，TF 的版本是 1.12。上面的代码会被编译成如下的 Kubernetes Pod 描述：

```yaml
container:
  ...
  resources:
    limits:
      cloud-tpus.google.com/v2: "8"
  metadata:
    annotations:
      tf-version.cloud-tpus.google.com: "1.12"
```

查看 [GKE TPU 指南](https://cloud.google.com/tpu/docs/kubernetes-engine-setup) 了解更多关于 TPU 的设置。

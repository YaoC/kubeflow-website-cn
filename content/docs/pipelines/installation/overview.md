+++
title = "Kubeflow Pipelines 的安装选项"
description = "部署 Kubeflow Pipelines 的几种方式概述"
weight = 10
+++

Kubeflow Pipelines 提供了几种安装方式可供选择。
本页面介绍了这几种安装选项以及它们各自可用的功能。

* Kubeflow Pipelines [独立](#standalone)部署。
* Kubeflow Pipelines 作为 [Kubeflow 部署的一部分](#full-kubeflow)。
* **Alpha**: [GCP 托管的 ML Pipelines](#marketplace)。

<a id="standalone"></a>
## 独立部署 Kubeflow Pipelines

该选项将 Kubeflow Pipelines 部署到本地或者云端的 Kubernetes 集群中，不包含其它 Kubeflow 的组件。
部署独立的 Kubeflow Pipelines 只需要使用 kustomize 清单。
这使得你可以很轻易地自定义自己的部署并将它集成到现有的 Kubernetes 集群中。 

安装指南
: [Kubeflow Pipelines 独立部署指南](/docs/pipelines/installation/standalone-deployment/)

接口
: 
  * Kubeflow Pipelines UI
  * Kubeflow Pipelines SDK
  * Kubeflow Pipelines API


特定功能的注意事项
: 部署完成后你的 Kubernetes 集群中只有 Kubeflow Pipelines。 
  集群中没有其它的 Kubeflow 组件。 
  例如集群中没有 Jupyter Notebook， 你可以使用本地或者由诸如 [AI Platform 
  Notebooks](https://cloud.google.com/ai-platform/notebooks/docs/) 等一些云服务托管的 notebook。

<a id="full-kubeflow"></a>
## Kubeflow 整体部署

该选项将 Kubeflow Pipelines 作为 Kubeflow 整体安装的一部分部署到个人电脑、本地服务器或者云端。

安装指南
: [Kubeflow 安装指南](/docs/started/getting-started/)

接口
:
  * Kubeflow UI
  * Kubeflow UI 内部或者外部的 Kubeflow Pipelines UI
  * Kubeflow Pipelines SDK
  * Kubeflow Pipelines API
  * 其它 Kubeflow APIs

特定功能的注意事项
: 部署完成后你的 Kubernetes 集群中包括了所有的 
  [Kubeflow 组件](/docs/components/)。
  例如你可以通过 
  [Kubeflow 内置的 Jupyter notebook 服务](/docs/notebooks/) Kubeflow 集群中创建一个或多个 notebook 服务器。

<a id="marketplace"></a>
## GCP 托管的 ML Pipelines

{{% alert title="Alpha release" color="warning" %}}
<p>GCP 托管的 ML Pipelines 现在还处于 <b>Alpha</b> 阶段，仅提供有限的支持。Kubeflow 团队欢迎你的任何反馈，
  尤其是可用性方面的。发送邮件到
  <a href="mailto:kfp-mkp-alpha-feedback@googlegroups.com">kfp-mkp-alpha-feedback@googlegroups.com</a> 可以获取到 Alpha 版本。
  你可以到
  <a href="https://github.com/kubeflow/pipelines/issues">Kubeflow Pipelines 
  issue tracker</a> 提问或是参与讨论。</p>
{{% /alert %}}

该选项从 GCP Marketplace 获取 Kubeflow Pipelines 并将它部署到 Google Kubernetes Engine (GKE) 中。
你可以将 Kubeflow Pipelines 部署到现有的或是部署时新创建的 GKE 集群中并在 GCP 中管理你的集群。 

安装指南
: [从 Google Cloud
  Marketplace 中获取并部署 Kubeflow Pipelines](https://github.com/kubeflow/pipelines/blob/master/manifests/gcp_marketplace/guide.md)

接口
: 
  * 用于管理 Kubeflow Pipelines 集群的 GCP 控制台或是其它 GCP 
    服务
  * 从 GCP 控制台的 **Open Pipelines Dashboard** 链接跳转到的 
    Kubeflow Pipelines UI
  * Cloud Notebooks 中的 Kubeflow Pipelines SDK


特定功能的注意事项
: 部署完成后你的 Kubernetes 集群中只有 Kubeflow Pipelines。 
  集群中没有其它的 Kubeflow 组件。 
  例如集群中没有 Jupyter Notebook，但你可以使用 [AI Platform 
  Notebooks](https://cloud.google.com/ai-platform/notebooks/docs/)。

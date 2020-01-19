+++
title = "流水线"
description = "Kubeflow Pipelines 中的流水线概念概述"
weight = 10
+++

*流水线* 是一个机器学习（ML）工作流的描述，包括该工作流中所有的组件以及这些组件如何以 [图](/docs/pipelines/concepts/graph/) 的形式相互关联。流水线的配置包括运行所需的输入（参数）定义以及每个组件的输入和输出。

当你运行一个流水线时，系统会启动一个或多个与你的工作流（流水线）中的步骤（组件）相对应的 Kubernetes Pods。Pods 会启动 Docker 容器，然后容器依次启动你的程序。

开发完你的流水线后，你可以在 Kubeflow Pipelines UI 上上传并共享它。

## 下一步

* 阅读 [Kubeflow Pipelines 概览](/docs/pipelines/pipelines-overview/)。
* 按照 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/) 部署 Kubeflow
  并在 Kubeflow Pipelines UI 中直接运行一个示例。
  
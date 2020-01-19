+++
title = "组件"
description = "Kubeflow Pipelines 中组件的概念概述"
weight = 20
+++

*流水线组件* 是代表机器学习工作流（流水线）中一个步骤的独立代码集，例如数据处理、数据变换或模型训练等。组件和函数类似，在组件里有名称、参数、返回值和主体。

## 组件代码

每个组件的代码包括以下几部分：

* **客户端代码：** 与端点交互提交作业的代码。例如，与 Google Dataproc API 进行交互以提交 Spark 作业的代码。

* **运行时代码：** 执行真正工作，通常在集群中运行的代码。例如，将原始数据转换为预处理数据的 Spark 代码。

请注意客户端代码和运行时代码的命名规约 —— 以一个名为 “mytask” 的任务为例：

* `mytask.py` 程序包含客户端代码。
* `mytask` 目录包含所有运行时代码。

## 组件定义

YAML 格式的组件规范描述了 Kubeflow Pipelines 系统中的组件。组件定义包含以下部分：

* **元数据：** 名称、描述等。
* **接口：** 输入/输出规范（名称、类型、描述、缺省值等）。
* **实现：** 在给定一组输入组件的参数值的情况下组件如何运行的规范。实现部分还描述了一旦组件运行完毕，如何从组件获取输出值。

有关组件的完整定义，请参阅 [组件规范](/docs/pipelines/reference/component-spec/)。

## 容器化组件

你必须把你的组件打包成 [Docker 镜像](https://docs.docker.com/get-started/)。组件代表了容器内的特定程序或入口点。

流水线中的每个组件都独立执行。这些组件不能在同一进程中运行，也不能直接共享内存中的数据。你必须序列化（字符串或文件的形式）所有需要在组件间传递的数据块以便数据能够在分布式网络中传输。此后你必须反序列化数据以供下游组件使用。

## 下一步

* 阅读 [Kubeflow Pipelines 概览](/docs/pipelines/pipelines-overview/)。
* 按照 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/) 部署 Kubeflow
  并在 Kubeflow Pipelines UI 中直接运行一个示例。
* 构建你自己的 [组件和流水线](/docs/pipelines/sdk/build-component/)。
* 构建一个 [可复用的组件](/docs/pipelines/sdk/component-development/) 在多个流水线中共享。
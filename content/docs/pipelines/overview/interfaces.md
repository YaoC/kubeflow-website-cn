+++
title = "Pipelines 接口简介"
description = "你与 Kubeflow Pipelines 系统交互的几种方法"
weight = 20
+++

本文档介绍了 Kubeflow Pipelines 的接口。通过这些接口你可以在 Kubeflow Pipelines
上构建并运行机器学习（ML）工作流。

## 用户界面（UI）

你可以在 Kubeflow UI 里通过点击 **Pipeline Dashboard** 进入 Kubeflow Pipelines UI。Kubeflow Pipelines UI 如下图：
  <img src="/docs/images/pipelines-ui.png" 
    alt="Pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

在 Kubeflow Pipelines UI 中，您可以执行以下任务：

* 通过运行一个或多个预置的示例快速尝试流水线。
* 将流水线以压缩文件的形式上传。流水线可以是你已经构建好的（查看如何 [构建一个流水线](/docs/pipelines/sdk/build-component/#compile-the-pipeline) ）或是他人分享给你的。
* 创建 *实验* 将你的流水线的一个或多个 run 分组。查看 [实验的定义](/docs/pipelines/overview/concepts/experiment/)。
* 在实验中创建并启动一个 *run*。一个 run 是流水线的一次执行。查看 [run 的定义](/docs/pipelines/overview/concepts/run/)。
* 浏览你的流水线 run 的配置、图和输出。
* 比较某个实验中的一个或多个 run 的结果。
* 通过创建一个可重复的 run 来按计划执行多个 run。

查看 [快速入门指南](/docs/pipelines/pipelines-quickstart/) 获取更多有关访问 Kubeflow Pipelines UI 和运行示例的信息。

构建流水线组件时，你可以让输出的信息在 UI 上显示。请参阅有关 [导出指标](/docs/pipelines/sdk/pipelines-metrics/) 和 [在 UI 中可视化结果](/docs/pipelines/sdk/output-viewer/) 的指南。

## Python SDK

Kubeflow Pipelines SDK 提供了一些可用于指定并运行你的机器学习工作流的 Python 包。

查看 [Kubeflow Pipelines 
SDK 简介](/docs/pipelines/sdk/sdk-overview/) 获取关于使用 SDK 构建流水线组件和流水线的几种方式的概述。

## REST API

Kubeflow Pipelines API 对于持续集成/部署系统很有用，比如你可以将流水线的执行嵌入到 shell 脚本或是其它系统中。例如你可能会希望在新数据到来时触发一个流水线。

请参阅 [Kubeflow Pipelines API 参考文档](/docs/pipelines/reference/api/kubeflow-pipeline-api-spec/)。

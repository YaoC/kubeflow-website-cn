+++
title = "Run 和可重复的 Run"
description = "Kubeflow Pipelines 中 run 的概念概述"
weight = 50
+++

*run* 是流水线的一次独立的执行。Run 中包含了你尝试的所有实验的不可变的记录，同时它被设计成独立的以便可以复现。你可以通过在 Kubeflow Pipelines UI 上查看 run 的详细信息来跟踪它的进度，这些信息包括运行时的图、输出构件和 run 的每一阶段的日志。 

<a id=recurring-run>

*可重复的 run* 是流水线的一个可以重复运行的 run。可重复的 run 的配置包括了一个指定了所有参数的流水线副本和一个 [run 触发器](/docs/pipelines/overview/concepts/run-trigger/)。你可以在任意实验中启动一个可重复的 run，之后它会定期启动一个据具有相同配置的 run。你可以在 Kubeflow Pipelines UI 中启用/禁用可重复的运行。你还可以指定 run 的最大并发数来限制并行启动的 run 的数量。这对会运行很长时间同时又会被频繁触发运行的流水线很有帮助。

## 下一步

* 阅读 [Kubeflow Pipelines 概览](/docs/pipelines/pipelines-overview/)。
* 按照 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/) 部署 Kubeflow
  并在 Kubeflow Pipelines UI 中直接运行一个示例。
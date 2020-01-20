+++
title = "Run 触发器"
description = "Kubeflow Pipelines 中 run 触发器的概念概述"
weight = 60
+++

*run 触发器* 是一个告诉系统可重复的 run 配置何时产生一个新的 run 的标识。run 触发器包括以下几种类型：

* 周期型：基于一定的时间间隔调度 run（例如：每 2 小时或每 45 分钟）。
* 定时任务型：通过指定 `cron` 语义来调度 run。

## 下一步

* 阅读 [Kubeflow Pipelines 概览](/docs/pipelines/pipelines-overview/)。
* 按照 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/) 部署 Kubeflow
  并在 Kubeflow Pipelines UI 中直接运行一个示例。
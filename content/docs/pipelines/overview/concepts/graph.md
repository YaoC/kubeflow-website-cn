+++
title = "图"
description = "Kubeflow Pipelines 中图的概念概述"
weight = 30
+++

*图* 是 Kubeflow Pipelines UI 中流水线运行时执行过程的图形化表示。图显示了流水线的 run 中已经执行完或正在执行的步骤，箭头表示每个步骤代表的流水线组件之间的父子关系。只要 run 开始后就可以查看图。图中的每个节点对应于流水线中的一个步骤，并相应地进行了标记。

下面的屏幕截图显示了流水线图的一个例子：

<img src="/docs/images/pipelines-xgboost-graph.png" 
  alt="XGBoost results on the pipelines UI"
  class="mt-3 mb-3 border border-info rounded">

每个节点的右上角都有一个表示状态的图标：运行、成功、失败或跳过。（当节点的父节点包含条件子句时，该节点可以跳过。）

## 下一步

* 阅读 [Kubeflow Pipelines 概览](/docs/pipelines/pipelines-overview/)。
* 按照 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/) 部署 Kubeflow
  并在 Kubeflow Pipelines UI 中直接运行一个示例。
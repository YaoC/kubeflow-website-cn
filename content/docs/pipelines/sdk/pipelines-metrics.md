+++
title = "流水线指标"
description = "导出并可视化流水线指标"
weight = 90
+++

本文档将向你展示如何从 Kubeflow Pipelines 组件中导出指标。获取更多关于如何构建组件的详细信息，请查看
[构建你自己的组件](/docs/pipelines/sdk/build-component/) 指南。
 
## 指标概述

Kubeflow Pipelines 支持导出标量指标。你可以通过将指标列表写入本地文件来描述模型的性能。
流水线代理会将该本地文件作为运行时指标上传。你可以在 Kubeflow Pipelines UI 中的特定实验的
**Runs** 页面中以可视化形式查看上传的指标。
 
## 导出指标文件

要启用指标功能，你的程序必须输出一个名为 `/mlpipeline-metrics.json` 的文件。例如：

```Python
  accuracy = accuracy_score(df['target'], df['predicted'])
  metrics = {
    'metrics': [{
      'name': 'accuracy-score', # The name of the metric. Visualized as the column name in the runs table.
      'numberValue':  accuracy, # The value of the metric. Must be a numeric value.
      'format': "PERCENTAGE",   # The optional format of the metric. Supported values are "RAW" (displayed in raw format) and "PERCENTAGE" (displayed in percentage format).
    }]
  }
  with file_io.FileIO('/mlpipeline-metrics.json', 'w') as f:
    json.dump(metrics, f)
```

查看
[完整的示例](https://github.com/kubeflow/pipelines/blob/master/components/local/confusion_matrix/src/confusion_matrix.py)。

指标文件需要满足以下要求：

* 文件路径必须为 `/mlpipeline-metrics.json`。
* `name` 必须匹配模式 `^[a-z]([-a-z0-9]{0,62}[a-z0-9])?$`。
* `numberValue` 必须为数值。
* `format` 必须是 `PERCENTAGE`，`RAW`，或是不设置。

## 查看指标

要查看指标的可视化视图：

1. 在 Kubeflow Pipelines UI 中打开 **Experiments** 页面。
1. 点击一个你的实验。打开的 **Runs** 页面中显示了头两个指标，头两个指标是什么首先由普遍程度（也就是指标计数的最高值）决定，其次是指标名称。
  指标作为每个 run 的一列显示。
  
以下示例显示了一个实验中的两个 run 的 **accuracy-score** 和
**roc-auc-score** 指标：

<img src="/docs/images/taxi-tip-run-scores.png" 
  alt="Metrics from a pipeline run"
  class="mt-3 mb-3 border border-info rounded">

## 下一步

通过 [把元数据写到输出查看器](/docs/pipelines/metrics/output-viewer/) 来可视化你的组件输出。

+++
title = "Pipelines 快速入门"
description = "Kubeflow Pipelines 入门"
weight = 10
+++

参照本指南你可以在 Kubeflow Pipelines 中快速创建一个简单的流水线。
如果需要更加深入了解 Kubeflow Pipelines， 请查看[端到端的教程](/docs/gke/pipelines-tutorial/)。

* 你可以在 Kubeflow Pipelines 的用户界面上看到几个安装时附带的示例，本指南将向你展示如何使用其中的一个示例。因此本指南也可以看作是 Kubeflow Pipelines UI 的介绍。
* 端到端的教程介绍了如何编写、编译一个流水线然后把它上传到 Kubeflow Pipelines 运行的全过程。

## 部署 Kubeflow 然后进入流水线页面

按照以下步骤部署 Kubeflow 后打开流水线仪表盘页面

1. 参照该指南 [在 GCP 中部署 Kubeflow](/docs/gke/deploy/)。

    {{% pipelines-compatibility %}} 

1. 根据 Kubeflow 安装指南启动 Kubeflow 后， 通过 `https://<deployment-name>.endpoints.<project>.cloud.goog/` 形式的 URL 进入 Kubeflow UI。 Kubeflow UI 如下图所示：
  <img src="/docs/images/central-ui.png" 
    alt="Kubeflow UI"
    class="mt-3 mb-3 border border-info rounded">

    如果在部署 Kubeflow 时跳过了 Cloud IAP 选项或是还没有暴露 Kubeflow 端点，那么你需要通过 `kubectl` 配置端口转发后才能访问 Kubeflow：
    
    1. 如果没有安装 `kubectl` 则通过以下命令进行安装： 
      `gcloud components install kubectl`。 获取更多相关信息请查看 
      [`kubectl` 
      文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。

    1. 执行 ```kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80``` 后访问 `http://localhost:8080/`。

1. 点击 **Pipelines** 按钮进入如下图所示的流水线主界面：
  <img src="/docs/images/pipelines-ui.png" 
    alt="Pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

## 运行基础的流水线

流水线主界面中预置的几个示例可让你快速上手。以下步骤将展示如何运行一个基础的示例。该示例仅包含一些 Python 操作，并未涉及机器学习（ML）的内容。

1. 在流水线页面点击 **\[Sample\] Basic - Parallel Execution** 示例的名字：
  <img src="/docs/images/click-pipeline-sample.png" 
    alt="Pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

1. 点击 **Create experiment**：
  <img src="/docs/images/pipelines-start-experiment.png" 
    alt="Starting an experiment on the pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

1. 根据提示先创建一个 **实验**，然后再创建一个 **run**。该示例提供了用户所需的所有参数的缺省值。下面的截图假定你已经创建了一个名叫 _My experiment_ 的实验后正在创建名叫 _My first run_ 的 run：
  <img src="/docs/images/pipelines-start-run.png" 
    alt="Creating a run on the pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

1. 点击 **Start** 创建 run。
1. 在实验仪表盘上点击 run 的名字：
  <img src="/docs/images/pipelines-experiments-dashboard.png" 
    alt="Experiments dashboard on the pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

1. 点击流水线图的各个组件和页面中的其他元素来查看流水线图和 run 的其它内容：
  <img src="/docs/images/pipelines-basic-run.png" 
    alt="Run results on the pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

你可以在 Kubeflow Pipelines 代码仓库中查看 [the basic parallel join sample 的源码](https://github.com/kubeflow/pipelines/blob/master/samples/core/parallel_join/parallel_join.py)。

## 运行机器学习流水线

本节介绍如何运行流水线用户界面中提供的 XGBoost 示例。与上述基础示例不同，XGBoost 示例中包含了一些机器学习组件。
在运行本示例之前你需要先创建好一些 GCP 服务供其使用。

按照以下步骤创建必要的 GCP 服务并运行示例：

1. 除了 Kubeflow 需要用到的基本的 GCP API 外（参考
  [GCP 安装指南](/docs/gke/deploy/project-setup)）流水线还需要以下 API：

    * [Cloud Storage](https://console.cloud.google.com/apis/library/storage-component.googleapis.com)
    * [Dataproc](https://console.cloud.google.com/apis/library/dataproc.googleapis.com)

1. 创建一个 
  [Cloud Storage bucket](https://console.cloud.google.com/storage/create-bucket) 
  来存储流水线运行的结果。

  * *bucket 名称* 必须保证在 Cloud Storage 中唯一。
  * 流水线中每创建一个 run， Kubeflow 就在 bucket 中创建一个唯一的目录，因此先前完成的 run 的输出结果并不会被之后完成的 run 的输出结果所覆盖。

1. 在流水线页面点击
  **\[Sample\] ML - XGBoost - Training with Confusion Matrix** 示例的名字：
  <img src="/docs/images/click-xgboost-sample.png" 
    alt="XGBoost sample on the pipelines UI"
    class="mt-3 mb-3 border border-info rounded">

1. 点击 **Create experiment**.
1. 根据提示创建一个 **实验**，之后再创建一个 **run**。
  输入以下 **run parameters**:

  * **output:** 先前创建的用于存储流水线 run 输出结果的 Cloud Storage bucket。
  * **project:** 你的 GCP 项目 ID。

    该示例已经提供了其它参数的值：

  * region: 存储训练和评估用的数据的 GCP 地理区域。
  * train-data: 训练数据的 Cloud Storage 路径。
  * eval-data: 评估数据的 Cloud Storage 路径。
  * schema: 描述存储训练和评估数据的 CSV 文件格式的 JSON 文件的 Cloud Storage 路径。
  * target: 目标变量的列名。
  * rounds: XGBoost 训练的迭代轮数。
  * workers: 分布式训练的节点数。
  * true-label: 模型输出标签对应的文本信息所在的列。

    下面的屏幕部分截图展示了 run 的参数，其中包括两个用户必须提供的参数：
    <img src="/docs/images/pipelines-start-xgboost-run.png" 
      alt="Starting the XGBoost run on the pipelines UI"
      class="mt-3 mb-3 border border-info rounded">

1. 点击 **Start** 创建 run。
1. 在实验仪表盘上点击 run 的名字。
1. 点击流水线图的各个组件和页面中的其他元素来查看流水线图和 run 的其它内容：
    <img src="/docs/images/pipelines-xgboost-graph.png" 
      alt="XGBoost results on the pipelines UI"
      class="mt-3 mb-3 border border-info rounded">

你可以在 [Kubeflow Pipelines 
代码仓库](https://github.com/kubeflow/pipelines/tree/master/samples/core/xgboost_training_cm) 中查看 XGBoost 训练示例的源代码。

## 清理你的 GCP 环境

本指南使用了付费的 GCP 组件。在你按照本指南运行完示例后需要按照以下步骤清理相关资源以尽可能降低费用：

1. 登录 [Deployment Manager](https://console.cloud.google.com/dm) 删除你的部署和相关资源。
1. 在检查过流水线的输出后删除 [Cloud Storage bucket](https://console.cloud.google.com/storage)。

## 后续

* 了解更多 Kubeflow Pipelines 的 [重要概念](/docs/pipelines/concepts/)。
* 参照 [端到端教程](/docs/gke/pipelines-tutorial/) 用 MNIST 数据集训练一个机器学习模型。
* 本页面现你展示了如何运行 Kubeflow Pipelines UI 中预置的示例。
  接下来你可以使用教程并结合 [Kubeflow Pipelines 示例](/docs/pipelines/tutorials/build-pipeline/) 来在 notebook 中运行流水线或是用代码编译并运行示例。
* 使用 [Kubeflow Pipelines 
  SDK](/docs/pipelines/sdk/sdk-overview/) 来运行你自己的机器学习流水线。

+++
title = "Kubeflow Pipelines 概览"
description = "理解 Kubeflow Pipelines 的目标和主要概念"
weight = 10
+++

Kubeflow Pipelines 是一个用于构建和部署可移植、可扩展，基于 Docker 容器的机器学习（ML）工作流的平台。

## 快速入门

按照 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart) 运行你的第一个流水线。

## 什么是 Kubeflow Pipelines?

Kubeflow Pipelines 平台包括：

* 一个用于管理跟踪实验、作业和 run 的交互界面。
* 一个多阶段机器学习工作流调度的引擎。
* 一个用于定义、操作流水线和组件的SDK。
* 使用 SDK 与系统交互的 Notebooks。

以下是 Kubeflow Pipelines 的目标：

* 端到端编排：提供并简化机器学习流水线的编排。
* 轻松进行实验：让你轻松尝试各种想法和技术，同时管理各种试验/实验。
* 轻松复用：让你能够复用组件和流水线以快速创建端到端解决方案，而不必每次都进行重建。

在 
[Kubeflow v0.1.3 或更高的版本](https://github.com/kubeflow/pipelines/releases/tag/0.1.3) 里，
Kubeflow Pipelines 是 Kubeflow 核心组件之一。 它在 Kubeflow 部署时自动部署。

{{% pipelines-compatibility %}}

## 什么是流水线？

_流水线_ 是机器学习工作流的描述，包括了工作流中所有的组件以及它们如何以图形形式组合。（下面的截图展示了一个流水线图的示例。）
流水线包括了运行所需的输入（参数）定义以及每个组件的输入和输出。

开发完你的流水线后你可以通过 Kubeflow Pipelines UI 上传并分享它。

_流水线组件_ 是一组独立的用户代码打包成 Docker [镜像](https://docs.docker.com/get-started/)，作为在流水线中执行的一个步骤。例如，一个组件可以负责数据预处理，数据变换或是模型训练等。

请参阅 [流水线](/docs/pipelines/concepts/pipeline/)
和 [组件](/docs/pipelines/concepts/component/) 的概念指南。

## 流水线示例

下面的屏幕截图和代码展示了 `xgboost-training-cm.py` 流水线，它使用 CSV 格式的结构化数据创建了一个 XGBoost 模型。
你可以在 [GitHub](https://github.com/kubeflow/pipelines/tree/master/samples/core/xgboost_training_cm) 上查看该流水线的源代码和其它相关信息。

### 流水线的运行时执行图

下面的屏幕截图展示了 Kubeflow Pipelines UI 中流水线示例的运行时执行图：

<img src="/docs/images/pipelines-xgboost-graph.png" 
  alt="XGBoost results on the pipelines UI"
  class="mt-3 mb-3 border border-info rounded">

### 定义流水线的 Python 代码

以下是定义 `xgboost-training-cm.py` 流水线的 Python 代码中的一部分。你可以在 [GitHub](https://github.com/kubeflow/pipelines/tree/master/samples/core/xgboost_training_cm) 上查看完整的代码。

```python
@dsl.pipeline(
    name='XGBoost Trainer',
    description='A trainer that does end-to-end distributed training for XGBoost models.'
)
def xgb_train_pipeline(
    output='gs://your-gcs-bucket',
    project='your-gcp-project',
    cluster_name='xgb-%s' % dsl.RUN_ID_PLACEHOLDER,
    region='us-central1',
    train_data='gs://ml-pipeline-playground/sfpd/train.csv',
    eval_data='gs://ml-pipeline-playground/sfpd/eval.csv',
    schema='gs://ml-pipeline-playground/sfpd/schema.json',
    target='resolution',
    rounds=200,
    workers=2,
    true_label='ACTION',
):
    output_template = str(output) + '/' + dsl.RUN_ID_PLACEHOLDER + '/data'

    # Current GCP pyspark/spark op do not provide outputs as return values, instead,
    # we need to use strings to pass the uri around.
    analyze_output = output_template
    transform_output_train = os.path.join(output_template, 'train', 'part-*')
    transform_output_eval = os.path.join(output_template, 'eval', 'part-*')
    train_output = os.path.join(output_template, 'train_output')
    predict_output = os.path.join(output_template, 'predict_output')

    with dsl.ExitHandler(exit_op=dataproc_delete_cluster_op(
        project_id=project,
        region=region,
        name=cluster_name
    )):
        _create_cluster_op = dataproc_create_cluster_op(
            project_id=project,
            region=region,
            name=cluster_name,
            initialization_actions=[
              os.path.join(_PYSRC_PREFIX,
                           'initialization_actions.sh'),
            ],
            image_version='1.2'
        )

        _analyze_op = dataproc_analyze_op(
            project=project,
            region=region,
            cluster_name=cluster_name,
            schema=schema,
            train_data=train_data,
            output=output_template
        ).after(_create_cluster_op).set_display_name('Analyzer')

        _transform_op = dataproc_transform_op(
            project=project,
            region=region,
            cluster_name=cluster_name,
            train_data=train_data,
            eval_data=eval_data,
            target=target,
            analysis=analyze_output,
            output=output_template
        ).after(_analyze_op).set_display_name('Transformer')

        _train_op = dataproc_train_op(
            project=project,
            region=region,
            cluster_name=cluster_name,
            train_data=transform_output_train,
            eval_data=transform_output_eval,
            target=target,
            analysis=analyze_output,
            workers=workers,
            rounds=rounds,
            output=train_output
        ).after(_transform_op).set_display_name('Trainer')

        _predict_op = dataproc_predict_op(
            project=project,
            region=region,
            cluster_name=cluster_name,
            data=transform_output_eval,
            model=train_output,
            target=target,
            analysis=analyze_output,
            output=predict_output
        ).after(_train_op).set_display_name('Predictor')

        _cm_op = confusion_matrix_op(
            predictions=os.path.join(predict_output, 'part-*.csv'),
            output_dir=output_template
        ).after(_predict_op)

        _roc_op = roc_op(
            predictions_dir=os.path.join(predict_output, 'part-*.csv'),
            true_class=true_label,
            true_score_column=true_label,
            output_dir=output_template
        ).after(_predict_op)

    dsl.get_pipeline_conf().add_op_transformer(
        gcp.use_gcp_secret('user-gcp-sa'))
```

### Kubeflow Pipelines UI 上的流水线输入数据

以下部分的屏幕截图展示了启动流水线的一次 run 的 Kubeflow Pipelines UI。你代码中的流水线定义决定了哪些参数会出现在 UI 表单中。流水线的定义里还可以设置参数的缺省值：

<img src="/docs/images/pipelines-start-xgboost-run.png" 
  alt="Starting the XGBoost run on the pipelines UI"
  class="mt-3 mb-3 border border-info rounded">

### 流水线的输出

下面的屏幕截图展示了在 Kubeflow Pipelines UI 上可见的流水线输出示例。

预测结果：

<img src="/docs/images/predict.png" 
  alt="Prediction output"
  class="mt-3 mb-3 p-3 border border-info rounded">

混淆矩阵：

<img src="/docs/images/cm.png" 
  alt="Confusion matrix"
  class="mt-3 mb-3 p-3 border border-info rounded">

接收者操作特征（ROC）曲线：

<img src="/docs/images/roc.png" 
  alt="ROC"
  class="mt-3 mb-3 p-3 border border-info rounded">

## 架构概览

<img src="/docs/images/pipelines-architecture.png" 
  alt="Pipelines architectural diagram"
  class="mt-3 mb-3 p-3 border border-info rounded">

从上层看，流水线的执行过程如下：

* **Python SDK**：你可以使用 Kubeflow
  Pipelines 领域特定语言（[DSL](https://github.com/kubeflow/pipelines/tree/master/sdk/python/kfp/dsl)）创建组件或者指定一个流水线。
* **DSL 编译器**：[DSL 编译器](https://github.com/kubeflow/pipelines/tree/master/sdk/python/kfp/compiler) 把你的流水线 Python 代码转变成一个静态配置（YAML）。
* **Pipeline Service**：你调用 Pipeline Service 通过静态配置创建一个流水线的 run。
* **Kubernetes 资源**：Pipeline Service 调用 Kubernetes API
  服务器创建必要的 Kubernetes 资源（[CRDs](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)）来运行流水线。  
* **编排控制器**：一组编排控制器执行 Kubernetes 资源（[CRDs](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)）指定的流水线执行需要完成的容器。容器在虚拟机上的 Kubernetes Pods 中执行。一个控制器的例子是 [**Argo Workflow**](https://github.com/argoproj/argo) 控制器，它可以编排任务驱动的工作流。
* **构件存储**：Pods 存储两种类型的数据： 

  * **元数据：** 实验、作业和 run 等。还有单独的标量指标，通常为了排序和过滤而汇总到一起。Kubeflow Pipelines 将元数据存储到 MySQL 数据库中。
  * **构件：** 流水线打包、视图等。还有像时间序列等大规模的指标，通常用于探究各个运行的 run 的性能或是调试。Kubeflow Pipelines 将这些构件存储在诸如 [Minio server](https://docs.minio.io/) 或 [Cloud Storage](https://cloud.google.com/storage/docs/) 的构件仓库中。

    MySQL 数据库和 Minio 服务器都由 Kubernetes [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)（PV）子系统支持。

* **持久化代理和机器学习元数据**：流水线持久化代理监听 Kubernetes 创建资源并将这些资源的状态持久化存储到机器学习元数据服务中。
  流水线持久化同时也记录已经执行的容器和它们的输入输出。输入输出既包括容器的参数也包括数据构件的 URL。
* **流水线 web 服务器**：流水线 web 服务器从各种服务收集数据并显示相关视图：当前运行中的流水线的列表，流水线执行的历史，数据构件的列表，各个运行的 run 的调试信息，各个运行的 run 的执行状态。

## 后续

* 根据 [pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart) 部署 Kubeflow 并从 Kubeflow Pipelines UI 直接运行一个流水线示例。
* 使用 [Kubeflow Pipelines SDK](/docs/pipelines/sdk/sdk-overview/) 构建机器学习流水线。
* 按照完整的指南使用 [the Kubeflow Pipelines 示例](/docs/pipelines/tutorials/build-pipeline/) 进行实验。
  

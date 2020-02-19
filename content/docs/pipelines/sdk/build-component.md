+++
title = "构建组件和流水线"
description = "构建你自己的流水线并将它添加到流水线中"
weight = 30
+++

本文档描述如何为 Kubeflow Pipelines 创建组件和如何将组件组合成流水线。要更轻松地开始，请尝试使用
[Kubeflow Pipelines 示例](/docs/pipelines/tutorials/build-pipeline/)。

## 流水线和组件概览

_流水线_ 是一个机器学习（ML）工作流的描述，包括了工作流中所有的组件以及它们如何协同工作。
流水线包括了运行流水线需要的输入（参数）的定义和每个组件的输入输出。

流水线 _组件_ 是流水线任务的实现。一个组件代表了工作流中的一个步骤。每个组件都接受一个或多个输入，
同时可能产生一个或多个输出。组件由接口（输入/输出）、实现（Docker 容器镜像和命令行参数）和元数据（名称、描述）构成。

获取更多信息，参考
[流水线](/docs/pipelines/concepts/pipeline/)
和 [组件](/docs/pipelines/concepts/component/) 概念指南。

## 在你开始之前

设置你的环境：

* 安装 [Docker](https://www.docker.com/get-docker)。
* 安装 [Kubeflow Pipelines SDK](/docs/pipelines/sdk/install-sdk/)。

本文档的示例来自 Kubeflow Pipelines 示例仓库的
[XGBoost Spark 流水线示例](https://github.com/kubeflow/pipelines/tree/master/samples/core/xgboost_training_cm)。

## 为每一个组件创建容器镜像

本节假设你已经创建好一个程序来执行机器学习工作流特定步骤所需的任务。比如，如果任务是要训练一个机器学习模型，
那么你必须要有一个程序来进行训练，例如一个
[训练 XGBoost 模型](https://github.com/kubeflow/pipelines/blob/master/components/deprecated/dataproc/train/src/train.py)
的程序。

创建一个打包了你的程序的 [Docker](https://docs.docker.com/get-started/) 容器镜像。 
参考上面提及的 XGBoost 模型训练示例程序的
[Docker file](https://github.com/kubeflow/pipelines/blob/master/components/deprecated/dataproc/train/Dockerfile)。
你还可以查看可复用组件的 Kubeflow Pipelines 仓库中通用的
[`build_image.sh`](https://github.com/kubeflow/pipelines/blob/master/components/build_image.sh) 脚本。

你的组件可以创建输出，下游组件可以将其作为输入。每个输出都必须是字符串并且容器镜像必须将每个输出写入单独的本地文本文件中。
例如，如果一个训练组件需要输出训练模型的路径，该组件将路径写入到一个本地文件中，例如是 `/output.txt`。在定义流水线（参见 [下面](#define-pipeline)）的
Python 类中你可以指定如何将本地文件的内容映射到组件输出。

## 创建 Python 函数包装你的组件

定义一个 Python 函数来描述与包含来流水线组件的 Docker 容器镜像的交互。例如，下面的 Python
函数描述了一个训练 XGBoost 模型的组件：

```python
def dataproc_train_op(
    project,
    region,
    cluster_name,
    train_data,
    eval_data,
    target,
    analysis,
    workers,
    rounds,
    output,
    is_classification=True
):
    if is_classification:
      config='gs://ml-pipeline-playground/trainconfcla.json'
    else:
      config='gs://ml-pipeline-playground/trainconfreg.json'

    return dsl.ContainerOp(
        name='Dataproc - Train XGBoost model',
        image='gcr.io/ml-pipeline/ml-pipeline-dataproc-train:ac833a084b32324b56ca56e9109e05cde02816a4',
        arguments=[
            '--project', project,
            '--region', region,
            '--cluster', cluster_name,
            '--train', train_data,
            '--eval', eval_data,
            '--analysis', analysis,
            '--target', target,
            '--package', 'gs://ml-pipeline-playground/xgboost4j-example-0.8-SNAPSHOT-jar-with-dependencies.jar',
            '--workers', workers,
            '--rounds', rounds,
            '--conf', config,
            '--output', output,
        ],
        file_outputs={
            'output': '/output.txt',
        }
    )

```

函数必须返回一个 dsl.ContainerOp，来自
[XGBoost Spark 流水线示例](https://github.com/kubeflow/pipelines/blob/master/samples/core/xgboost_training_cm/xgboost_training_cm.py)。

注意：

* 每个组件都必须继承自
  [`dsl.ContainerOp`](https://github.com/kubeflow/pipelines/blob/master/sdk/python/kfp/dsl/_container_op.py)。
* 上面的 `dsl.ContainerOp` 构造器使用的 `参数` 列表的值必须是 Python 标量类型（例如 `str` 和 ` int`）或
  [`dsl.PipelineParam`](https://github.com/kubeflow/pipelines/blob/master/sdk/python/kfp/dsl/_pipeline_param.py)
  类型。每一个 `dsl.PipelineParam` 表示一个值通常只有在运行时才知道的参数。该值既可以在流水线运行时由用户提供，也可以接受上游组件的输出。
* 尽管每个 `dsl.PipelineParam` 的值只有在运行时才有效，你仍然可以通过 `%s` 变量替换来将内联形式参数作为 `实参` 使用。运行时实参中包含了形参的值。
* `file_outputs` 是标签和本地文件路径的映射。在上述示例中，`/output.txt` 的内容包括了组件的字符串输出。如果要在在代码中引用输出：

    ```python
    op = dataproc_train_op(...)
    op.outputs['label']
    ```

    如果仅有一个输出那么你也可以使用 `op.output`。

<a id="define-pipeline"></a>
## 将流水线定义为 Python 函数

你必须将每个流水线描述成一个 Python 函数。例如：

```python
@dsl.pipeline(
  name='XGBoost Trainer',
  description='A trainer that does end-to-end distributed training for XGBoost models.'
)
def xgb_train_pipeline(
    output,
    project,
    region='us-central1',
    train_data='gs://ml-pipeline-playground/sfpd/train.csv',
    eval_data='gs://ml-pipeline-playground/sfpd/eval.csv',
    schema='gs://ml-pipeline-playground/sfpd/schema.json',
    target='resolution',
    rounds=200,
    workers=2,
    true_label='ACTION',
)
```

注意：

* **@dsl.pipeline** 是必须的装饰，包含 `名称` 和 `描述` 属性。
* 输入参数在 Kubeflow Pipelines UI 中显示为流水线参数。作为 Python 规则，位置参数在前，接着是关键字参数。
* 每个函数参数的类型都是
  [`dsl.PipelineParam`](https://github.com/kubeflow/pipelines/blob/master/sdk/python/kfp/dsl/_pipeline_param.py)。
  缺省值均为该类型。缺省值显示在 Kubeflow Pipelines UI 中，但是用户可以覆盖它们。


查看
[XGBoost Spark 流水线示例](https://github.com/kubeflow/pipelines/blob/master/samples/core/xgboost_training_cm/xgboost_training_cm.py)
的完整代码。

## 编译流水线

如上所述在 Python 中定义了流水线后，你必须先将流水线编译成中间表示形式，然后才能提交到 Kubeflow Pipelines 服务。
中间表示是一个被压缩成 `.tar.gz` 格式文件的 YAML 格式的工作流说明。

使用 `dsl-compile` 命令来编译你的流水线：

```bash
dsl-compile --py [path/to/python/file] --output [path/to/output/tar.gz]
```

## 部署流水线

通过 Kubeflow Pipelines UI 上传生成的 `.tar.gz`。参见指南
[UI 入门](/docs/pipelines/pipelines-quickstart)。

## 下一步

* 构建 [可复用组件](/docs/pipelines/sdk/component-development/) 来在多个流水线之间共享。
* 了解更多关于 
  [Kubeflow Pipelines 领域特定语言（DSL）](/docs/pipelines/sdk/dsl-overview/) 的内容，
  一组可以用来创建机器学习工作流的 Python 库。
* 查看如何 [从流水线中导出指标](/docs/pipelines/metrics/pipelines-metrics/)。
* 通过 [为输出查看器添加元数据](/docs/pipelines/metrics/output-viewer/) 来可视化你的组件输出。
* 直接从 Python 函数 [构建轻量级组件](/docs/pipelines/sdk/lightweight-python-components/) 以快速迭代。

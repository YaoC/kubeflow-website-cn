+++
title = "Pipelines SDK 简介"
description = "使用 SDK 来构建组件和流水线的概述"
weight = 10
+++

[Kubeflow Pipelines SDK](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.html)
提供了一系列的 Python 包，你可以通过它们来指定并运行机器学习（ML）工作流。*流水线*
是一个机器学习工作流的描述，它包括了构成工作流各个步骤的所有组件已经这些组件之间如何进行交互。

## SDK 包

Kubeflow Pipelines SDK 包括了以下包：

* [`kfp.compiler`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html)
  包括用于为流水线组件构建容器镜像的类和方法。该包包含但不限于以下方法：

  * `kfp.compiler.Compiler.compile` 将你的 Python DSL 代码编译成 Kubeflow Pipelines
    服务能够处理的静态配置（YAML 格式）。Kubeflow Pipelines 会将该静态文件转换成一系列
    Kubernetes 资源以供执行。

  * `kfp.compiler.build_docker_image` 根据 Dockerfile 构建容器镜像，然后将镜像推送到指定的 URI 处。
    在该方法的参数中，你需要提供一个包含了镜像规格的 Dockerfile 的路径和一个用于定位存储目标镜像的 URI（例如，容器仓库地址）。

  * `kfp.compiler.build_python_component` 根据一个 Python 函数来构建一个流水线组件对应的容器镜像，
    然后然后将镜像推送到指定的 URI 处。在该方法的参数中，你需要提供执行流水线组件工作的 Python 函数，一个
    Docker 镜像用作基础镜像，还有一个用于定位存储目标镜像的 URI（例如，容器仓库地址）。

* [`kfp.components`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.components.html)
  包括用于和流水线组件交互的类和方法。该包包含但不限于以下方法：

  * `kfp.components.func_to_container_op` 将 Python 函数转换成流水线组件并返回一个工厂函数。
    接下来你可以调用该工厂函数来构造一个会在容器中运行原始函数的流水线任务的实例（[`ContainerOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.ContainerOp)）。

  * `kfp.components.load_component_from_file` 从文件中加载流水线组件并返回一个工厂函数。
    接下来你可以调用该工厂函数来构建一个运行组件容器镜像的流水线任务的实例（[`ContainerOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.ContainerOp)）。

  * `kfp.components.load_component_from_url` 从 URL 加载流水线组件并返回一个工厂函数。
    接下来你可以调用该工厂函数来构建一个运行组件容器镜像的流水线任务的实例（[`ContainerOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.ContainerOp)）。

* [`kfp.dsl`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html)
  包含了定义流水线和组件并与之交互的领域特定语言（DSL）。该包包含但不限于以下方法、类和模块：

  * `kfp.dsl.ContainerOp` 表示一个由容器镜像实现的流水线任务（op）。
  * `kfp.dsl.PipelineParam` 表示一个可以从一个流水线组件传递到另一个流水线组件的流水线参数。参见
    [流水线参数](/docs/pipelines/sdk/parameters/) 指南。
  * `kfp.dsl.component` 是一个返回流水线组件的 DSL 函数的装饰器。([`ContainerOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.ContainerOp))。
  * `kfp.dsl.pipeline` 是一个返回流水线的 Python 函数的装饰器。
  * `kfp.dsl.python_component` 是一个添加流水线组件元数据到函数对象的 Python 函数的装饰器。
  * [`kfp.dsl.types`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.types.html)
    包含一系列 Kubeflow Pipelines SDK 定义的类型。这些类型包括像 `String`、 `Integer`、 `Float` 和 `Bool`
    这样的基本类型，也包括像 `GCPProjectID` 和 `GCRPath` 这样的领域特定类型。参见
    [DSL 静态类型检查](/docs/pipelines/sdk/static-type-checking) 指南。
  * [`kfp.dsl.ResourceOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.ResourceOp)
    表示让你直接操作 Kubernetes 资源（`create`、 `get`、 `apply`、...）的流水线任务（op）。
  * [`kfp.dsl.VolumeOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.VolumeOp)
    表示创建一个新的 `PersistentVolumeClaim`（PVC）的流水线任务（op）。它旨在让创建 `PersistentVolumeClaim` 的常见情况变得更快。
  * [`kfp.dsl.VolumeSnapshotOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.VolumeSnapshotOp)
    表示创建一个新的 `VolumeSnapshot` 的流水线任务（op）。它旨在让创建 `VolumeSnapshot` 的常见情况变得更快。
  * [`kfp.dsl.PipelineVolume`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.PipelineVolume)
    表示用于在流水线各步骤间传递数据的存储卷。`ContainerOp` 可以通过构造器的参数 `pvolumes` 或 `add_pvolumes()` 方法来挂载 `PipelineVolume`。

* [`kfp.Client`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.client.html)
  包含 [Kubeflow Pipelines API](/docs/pipelines/reference/api/kubeflow-pipeline-api-spec/)
  的 Python 客户端库。该包包含但不限于以下方法：

  * `kfp.Client.create_experiment` 创建一个流水线 [实验](/docs/pipelines/concepts/experiment/) 并返回一个实验对象。
  * `kfp.Client.run_pipeline` 运行一个流水线并返回一个 run 对象。

* [`kfp.notebook`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.notebook.html)

* [KFP extension modules](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.extensions.html)
  包括在特定平台使用 Kubeflow Pipelines 的类和方法。示例包括了本地的、Google Cloud Platform（GCP）、Amazon Web Services（AWS）和 Microsoft
  Azure 的实用工具功能。

## 安装 SDK

参见指南
[安装 Kubeflow Pipelines SDK](/docs/pipelines/sdk/install-sdk/)。

## 构建流水线和组件

本节总结了可以用 SDK 来构建流水线和组件的方式：

* [从现有的应用代码来构建组件](#standard-component-outside-app)
* [在应用代码中构建组件](#standard-component-in-app)
* [构建轻量级组件](#lightweight-component)
* [在流水线中使用预构建的，可复用的组件](#prebuilt-component)

后续的图解为以下概念之间的关系提供了概念性的指导：

* 你的 Python 代码
* 流水线组件
* Docker 容器镜像
* 流水线

<a id="standard-component-outside-app"></a>
### 从现有的应用代码来构建组件

本节介绍如何在 Python 应用的 *外部* 构建流水线和组件，即通过从现有的容器化应用程序构建建组件。
该技术在你已经有现成的应用程序的时候非常有用，例如你已经创建了一个 TensorFlow 应用，然后想在流水线中使用它。

<img src="/docs/images/pipelines-sdk-outside-app.svg" 
  alt="Creating components outside your application code"
  class="mt-3 mb-3 border border-info rounded">

以下是上图的更加详细的说明：

1. 编写应用程序代码，`my-app-code.py`。例如，编写代码来变换数据或者训练模型。

1. 创建打包了你的应用程序（`my-app-code.py`）的 [Docker](https://docs.docker.com/get-started/)
  容器镜像并将它上传到镜像仓库中。你可以使用 [Docker 命令行接口](https://docs.docker.com/engine/reference/commandline/cli/)
  或 Kubeflow Pipelines SDK 中的 [`kfp.compiler.build_docker_image` 方法](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html#kfp.compiler.build_docker_image) 来根据指定的 [Dockerfile](https://docs.docker.com/engine/reference/builder/)
  构建容器镜像。

1. 使用 Kubeflow Pipelines DSL 编写组件函数来定义流水线和组件的 Docker 容器之间的交互。你的组件函数必须返回一个
  [`kfp.dsl.ContainerOp`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.ContainerOp)。
  你可以使用 [`kfp.dsl.component` 装饰器](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.component)
  来在 DSL 编译器中启用 [静态类型检查](/docs/pipelines/sdk/static-type-checking)。在组件函数上添加 `@kfp.dsl.component` 
  注解就可以使用该装饰器：

    ```python
    @kfp.dsl.component
    def my_component(my_param):
      ...
      return kfp.dsl.ContainerOp(
        name='My component name',
        image='gcr.io/path/to/container/image'
      )
    ```

1. 使用 Kubeflow Pipelines DSL 编写流水线函数来定义流水线和其中的所有流水线组件。使用
  [`kfp.dsl.pipeline` 装饰器](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.pipeline)
  来根据流水线函数构建流水线。在流水线函数上添加 `@kfp.dsl.pipeline` 注解就能使用该装饰器：

    ```python
    @kfp.dsl.pipeline(
      name='My pipeline',
      description='My machine learning pipeline'
    )
    def my_pipeline(param_1: PipelineParam, param_2: PipelineParam):
      my_step = my_component(my_param='a')
    ```

1. 编译流水线来生成压缩的流水线 YAML 格式的定义。Kubeflow Pipelines 服务会将静态的配置转换成一系列 Kubernetes 资源以供执行。

    要编译流水线，你可以选择以下的选项：

    * 使用 
      [`kfp.compiler.Compiler.compile`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html#kfp.compiler.Compiler) 
      方法：

        ```python
        kfp.compiler.Compiler().compile(my_pipeline,  
          'my-pipeline.zip')
        ```

    * 或者，在命令行中使用 `dsl-compile` 命令。

        ```shell
        dsl-compile --py [path/to/python/file] --output my-pipeline.zip
        ```

1. 使用 Kubeflow Pipelines SDK 运行流水线：

    ```python
    client = kfp.Client()
    my_experiment = client.create_experiment(name='demo')
    my_run = client.run_pipeline(my_experiment.id, 'my-pipeline', 
      'my-pipeline.zip')
    ```

你还可以选择共享你的流水线，具体如下：

* 将流水线 zip 压缩文件上传到 Kubeflow Pipelines UI。更多关于 UI 的信息请参考
  [Kubeflow Pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/)。
* 将流水线 zip 压缩文件上传到共享存储仓库。参见
  [可重用组件和其它共享资源](/docs/examples/shared-resources/)。

{{% alert title="有关上述工作流的更多信息" color="info" %}}
获取更详细的说明，参见指南 [构建组件和流水线](/docs/pipelines/sdk/build-component/)。

关于示例，参见 GitHub 上的
[`xgboost-training-cm.py`](https://github.com/kubeflow/pipelines/blob/master/samples/core/xgboost_training_cm/xgboost_training_cm.py)
流水线示例。该流水线使用 CSV 格式的结构化数据创建了一个 XGBoost 模型。
{{% /alert %}}

<a id="standard-component-in-app"></a>
### 在你的应用代码中创建组件

本节描述如何在 Python 应用 *内部* 创建一个流水线组件，以此作为应用的一部分。因此创建组件的 DSL 代码运行在你的 Docker 容器内部。

<img src="/docs/images/pipelines-sdk-within-app.svg" 
  alt="Building components within your application code"
  class="mt-3 mb-3 border border-info rounded">

以下是上图的更加详细的说明：

1. 在一个 Python 函数中编写你的代码。例如编写转换数据或训练模型的代码：

    ```python
    def my_python_func(a: str, b: str) -> str:
      ...
    ```

1. 使用 [`kfp.dsl.python_component`
  装饰器](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.python_component)
  将你的 Python 函数转换成一个流水线组件。在你的函数上添加 `@kfp.dsl.python_component` 注解就能使用该装饰器：

    ```python
    @kfp.dsl.python_component(
      name='My awesome component',
      description='Come and play',
    )
    def my_python_func(a: str, b: str) -> str:
      ...
    ```

1. 使用 
  [`kfp.compiler.build_python_component`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html#kfp.compiler.build_python_component)
  为组件创建一个容器镜像。

    ```python
    my_op = compiler.build_python_component(
      component_func=my_python_func,
      staging_gcs_path=OUTPUT_DIR,
      target_image=TARGET_IMAGE)
    ```

1. 使用 Kubeflow Pipelines DSL 编写一个流水线函数来定义流水线和其中的所有组件。使用
  [`kfp.dsl.pipeline` 装饰器](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.pipeline)
  把流水线函数构建成流水线， 通过在你的流水线函数上添加 `@kfp.dsl.pipeline` 注解即可：

    ```python
    @kfp.dsl.pipeline(
      name='My pipeline',
      description='My machine learning pipeline'
    )
    def my_pipeline(param_1: PipelineParam, param_2: PipelineParam):
      my_step = my_op(a='a', b='b')
    ```

1. 编译流水线来生成压缩的流水线 YAML 格式的定义。Kubeflow Pipelines 服务会将静态的配置转换成一系列 Kubernetes 资源以供执行。
  
    要编译流水线，你可以选择以下的选项：

    * 使用 
      [`kfp.compiler.Compiler.compile`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html#kfp.compiler.Compiler) 
      方法：

        ```python
        kfp.compiler.Compiler().compile(my_pipeline,  
          'my-pipeline.zip')
        ```

    * 或者，在命令行中使用 `dsl-compile` 命令。

        ```shell
        dsl-compile --py [path/to/python/file] --output my-pipeline.zip
        ```

1. 使用 Kubeflow Pipelines SDK 来运行流水线：

    ```python
    client = kfp.Client()
    my_experiment = client.create_experiment(name='demo')
    my_run = client.run_pipeline(my_experiment.id, 'my-pipeline', 
      'my-pipeline.zip')
    ```

你还可以选择共享你的流水线，具体如下：

* 将流水线 zip 压缩文件上传到 Kubeflow Pipelines UI。更多关于 UI 的信息请参考
  [Kubeflow Pipelines 快速入门指南](/docs/pipelines/pipelines-quickstart/)。
* 将流水线 zip 压缩文件上传到共享存储仓库。参见
  [可重用组件和其它共享资源](/docs/examples/shared-resources/)。

{{% alert title="有关上述工作流的更多信息" color="info" %}}
有关上述工作流的示例，参见 GitHub 上标题为
[KubeFlow Pipelines 基本组件构建](https://github.com/kubeflow/pipelines/blob/master/samples/core/component_build/component_build.ipynb) 
的 Jupyter notebook。
{{% /alert %}}

<a id="lightweight-component"></a>
### 创建轻量级组件

本节描述如何创建不需要构建容器镜像的轻量级 Python 组件。轻量级组件简化了原型设计并加快了开发速度，尤其是在 Jupyter notebook 环境中。

<img src="/docs/images/pipelines-sdk-lightweight.svg" 
  alt="Building lightweight Python components"
  class="mt-3 mb-3 border border-info rounded">

以下是上图的更加详细的说明：

1. 在一个 Python 函数中编写你的代码。例如编写转换数据或训练模型的代码：

    ```python
    def my_python_func(a: str, b: str) -> str:
      ...
    ```

1. 使用 
  [`kfp.components.func_to_container_op`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.components.html#kfp.components.func_to_container_op)
  将你的 Python 函数转换成一个流水线组件：

    ```python
    my_op = kfp.components.func_to_container_op(my_python_func)
    ```

    或者，你可以将组件写入一个文件以便进行共享或在其它流水线中使用：

    ```python
    my_op = kfp.components.func_to_container_op(my_python_func, 
      output_component_file='my-op.component')
    ```

1. 如果你在上一步中像描述的那样将轻量级组件存储到一个文件中，使用
  [`kfp.components.load_component_from_file`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.components.html#kfp.components.load_component_from_file)
  加载组件：

    ```python
    my_op = kfp.components.load_component_from_file('my-op.component')
    ```

1. 1. 使用 Kubeflow Pipelines DSL 编写一个流水线函数来定义流水线和其中的所有组件。使用
  [`kfp.dsl.pipeline` 装饰器](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.pipeline)
  把流水线函数构建成流水线， 通过在你的流水线函数上添加 `@kfp.dsl.pipeline` 注解即可：

    ```python
    @kfp.dsl.pipeline(
      name='My pipeline',
      description='My machine learning pipeline'
    )
    def my_pipeline(param_1: PipelineParam, param_2: PipelineParam):
      my_step = my_op(a='a', b='b')
    ```

1. 编译流水线来生成压缩的流水线 YAML 格式的定义。Kubeflow Pipelines 服务会将静态的配置转换成一系列 Kubernetes 资源以供执行。
  
    要编译流水线，你可以选择以下的选项：

    * 使用
      [`kfp.compiler.Compiler.compile`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html#kfp.compiler.Compiler) 
      方法：

        ```python
        kfp.compiler.Compiler().compile(my_pipeline,  
          'my-pipeline.zip')
        ```

    * 或者，在命令行中使用 `dsl-compile` 命令。

        ```shell
        dsl-compile --py [path/to/python/file] --output my-pipeline.zip
        ```

1. 使用 Kubeflow Pipelines SDK 运行流水线：

    ```python
    client = kfp.Client()
    my_experiment = client.create_experiment(name='demo')
    my_run = client.run_pipeline(my_experiment.id, 'my-pipeline', 
      'my-pipeline.zip')
    ```

{{% alert title="有关上述工作流的更多信息" color="info" %}}
要获取更详细的说明，参见指南
[构建轻量级组件](/docs/pipelines/sdk/lightweight-python-components/)。

有关示例，参见 GitHub 上的
[轻量级 Python 组件 —— 
基础](https://github.com/kubeflow/pipelines/blob/master/samples/core/lightweight_component/lightweight_component.ipynb)
notebook。
{{% /alert %}}

<a id="prebuilt-component"></a>
### 在流水线中使用预先构建的、可复用的组件

可复用的组件是作者已经构建好的并可以供他人使用的组件。要在你的流水线中使用这样的组件，你需要定义了该组件的 YAML 文件。

<img src="/docs/images/pipelines-sdk-reusable.svg" 
  alt="Using prebuilt, reusable components in your pipeline"
  class="mt-3 mb-3 border border-info rounded">

以下是上图的更加详细的说明：

1. 找到定义了可复用组件的 YAML 文件。例如，查看
  [可复用组件和其它共享资源](/docs/examples/shared-resources/)。

1. 使用 
  [`kfp.components.load_component_from_url`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.components.html#kfp.components.load_component_from_url)
  加载组件：

    ```python
    my_op = kfp.components.load_component_from_url('https://path/to/component.yaml')
    ```

1. 使用 Kubeflow Pipelines DSL 编写一个流水线函数来定义流水线和其中的所有组件。使用
  [`kfp.dsl.pipeline` 装饰器](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.pipeline)
  把流水线函数构建成流水线， 通过在你的流水线函数上添加 `@kfp.dsl.pipeline` 注解即可：

    ```python
    @kfp.dsl.pipeline(
      name='My pipeline',
      description='My machine learning pipeline'
    )
    def my_pipeline(param_1: PipelineParam, param_2: PipelineParam):
      my_step = my_op(a='a', b='b')
    ```

1. 编译流水线来生成压缩的流水线 YAML 格式的定义。Kubeflow Pipelines 服务会将静态的配置转换成一系列 Kubernetes 资源以供执行。
  
    要编译流水线，你可以选择以下的选项：

    * 使用
      [`kfp.compiler.Compiler.compile`](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.compiler.html#kfp.compiler.Compiler) 
      方法：

        ```python
        kfp.compiler.Compiler().compile(my_pipeline,  
          'my-pipeline.zip')
        ```

    * 或者，在命令行中使用 `dsl-compile` 命令。

        ```shell
        dsl-compile --py [path/to/python/file] --output my-pipeline.zip
        ```

1. 使用 Kubeflow Pipelines SDK 运行流水线：

    ```python
    client = kfp.Client()
    my_experiment = client.create_experiment(name='demo')
    my_run = client.run_pipeline(my_experiment.id, 'my-pipeline', 
      'my-pipeline.zip')
    ```
{{% alert title="有关上述工作流的更多信息" color="info" %}}
关于示例，参见 GitHub 上的
[`xgboost-training-cm.py`](https://github.com/kubeflow/pipelines/blob/master/samples/core/xgboost_training_cm/xgboost_training_cm.py)
流水线示例。该流水线使用 CSV 格式的结构化数据创建了一个 XGBoost 模型。
{{% /alert %}}

## 下一步

* [使用流水线参数](/docs/pipelines/sdk/parameters/) 在组件之间传递数据。
* 学习如何 [在 DSL 中编写递归函数](/docs/pipelines/sdk/dsl-recursion)。
* 构建 [可复用组件](/docs/pipelines/sdk/component-development/) 以便在多个流水线中共享。
* 了解如何使用 DSL 来 [把动态操作 Kubernetes 资源作为流水线中的步骤](/docs/pipelines/sdk/manipulate-resources/)。

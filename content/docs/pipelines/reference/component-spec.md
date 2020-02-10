+++
title = "组件规范"
description = "Kubeflow Pipelines 组件的定义"
weight = 10
+++

本规范描述了 Kubeflow Pipelines 的容器组件数据模型。数据模型被序列化成 YAML 格式的文件以进行共享。

以下是组件定义的主要部分：

* **元数据：** 名称，描述，和其它元数据。
* **接口（输入和输出）：** 名称，类型，默认值。
* **实现：** 给定输入参数，如何运行组件。

## 组件规范示例

组件规范采用 YAML 形式的文件， `component.yaml`。以下是一个示例：

```
name: xgboost4j - Train classifier
description: Trains a boosted tree ensemble classifier using xgboost4j

inputs:
- {name: Training data}
- {name: Rounds, type: Integer, default: '30', help: Number of training rounds}

outputs:
- {name: Trained model, type: XGBoost model, help: Trained XGBoost model}

implementation:
  container:
    image: gcr.io/ml-pipeline/xgboost-classifier-train@sha256:b3a64d57
    command: [
      /ml/train.py,
      --train-set, {inputPath: Training data},
      --rounds,    {inputValue: Rounds},
      --out-model, {outputPath: Trained model},
    ]
```

查看真实 [组件规范](https://github.com/kubeflow/pipelines/search?q=filename%3Acomponent.yaml&unscoped_q=filename%3Acomponent.yaml) 的一些示例。

## 详细规范（ComponentSpec）

本节描述 [ComponentSpec](https://github.com/kubeflow/pipelines/blob/master/sdk/python/kfp/components/_structures.py)。

### 元数据

* `name`：组件的可读名称。
* `description`：组件的描述。
* `metadata`：标准对象的元数据：

    * `annotations`: 用于添加组件相关信息的字符串键值对。
        目前而言，当组件任务在 Kubernetes 上执行时注解会被成 Kubernetes 注解。目前的缺陷：键中不能包含超过一个斜杠符号（“/”）。更多信息请查看 [Kubernetes 用户指南](http://kubernetes.io/docs/user-guide/annotations)。
    * `labels`：已废弃。使用 `annotations`。

### 接口

* `input` 和 `output`：
    指定输入/输出及其属性的列表。每一个输入或输出包括以下属性：

    * `name`：输入/输出的可读名称。输入或输出各部分中的名称必须唯一，但是输出的名称可以和输入的相同。 
    * `description`：输入/输出的可读描述。
    * `default`：指定一个输入的缺省值。仅在输入中有效。
    * `type`：指定输入/输出的类型。类型用来给流水线作者作为提示，同时也可以在流水线系统/交互界面上验证参数和组件之间的连接。基本类型有 **String**，**Integer**，**Float**，和 **Bool**。请参阅 Kubeflow Pipelines SDK 定义的 [类型](https://github.com/kubeflow/pipelines/blob/master/sdk/python/kfp/dsl/types.py) 的完整列表。

### 实现

* `实现`：指定如何执行组件实例。有两种实现类型，`容器` 和 `图`。（后者不在本文档范围内。）将来我们会引入更多的实现方式比如 `守护程序` 或 `K8sResource`。

    * `container`：
        描述实现组件的 Docker 容器。Kubernetes [Container v1 spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/#container-v1-core)
        的一个轻量的子集。

        * `image`：Docker 镜像的名称。
        * `command`：入口点数组。如果未提供则使用 Docker 镜像的 ENTRYPOINT。数组中每一个元素是字符串或占位符。最常见的占位符有
            `{inputValue: 输入名称}`，`{inputPath: 输入名称}` 和 `{outputPath: 输出名称}`。
        * `args`：入口点参数。如果未提供则使用 Docker 镜像的 CMD。数组中每一个元素是字符串或占位符。最常见的占位符有
            `{inputValue: 输入名称}`，`{inputPath: 输入名称}` 和 `{outputPath: 输出名称}`。
        * `env`：在容器中设置的环境变量的映射。
        * `fileOutputs`：遗留属性，仅在容器始终将输出数据存储在一些硬编码的不可配置的本地位置的情况下需要。
            此属性指定一些输出和程序将输出数据写入文件对应的本地文件路径之间的映射。仅在组件有硬编码输出路径时需要。
            这样的容器需要通过修改程序或是添加一个包装脚本将输出拷贝到一个可配置的位置。否则该组件可能与将来的存储系统不兼容。

当你在流水线中使用一个组件时你可以设置其它所有的 Kubernetes 容器属性。

## 使用占位符作为命令行参数

### 从值中获取输入

`{inputValue: <输入名称>}` 占位符会被替换成输入参数的值：

* 在 `component.yaml` 中：
  
  ```yaml
  command: [program.py, --rounds, {inputValue: Rounds}]
  ```

* 在流水线代码中：

  ```python
  task1 = component1(rounds=150)
  ```

* 生成的命令行代码（显示已替换占位符的输入参数的值）：

  ```shell
  program.py --rounds 150
  ```

### 从文件中获取输入

系统会提前把 “输入名称” 输入需要的参数数据提前放置在一个文件中，`{inputPath: <输入名称>}` 占位符被替换成该（自动生成）本地文件的路径。

* 在 `component.yaml` 中：

  ```yaml
  command: [program.py, --train-set, {inputPath: training_data}]
  ```

* 在流水线代码中：

  ```python
  task2 = component1(training_data=some_task1.outputs['some_data'])
  ```

* 生成的命令行代码（占位符被替换为生成的路径）： 

  ```shell
  program.py --train-set /inputs/train_data/data
  ```


### 创建输出

`{outputPath: <Output name>}` 被替换成组件程序应该写入输出数据对应的一个 （生成的）本地文件路径。
路径的父目录可能存在也可能不存在。您的程序必须正确处理这两种情况。

* 在 `component.yaml` 中：

  ```yaml
  command: [program.py, --out-model, {outputPath: trained_model}]
  ```

* 在流水线代码中：

  ```python
  task1 = component1()
  # You can now pass `task1.outputs['trained_model']` to other components as argument.
  ```

* 生成的命令行代码（占位符被替换为生成的路径）：

  ```shell
  program.py --out-model /outputs/trained_model/data
  ```
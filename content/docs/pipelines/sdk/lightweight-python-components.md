+++
title = "构建轻量级 Python 组件"
description = "使用 Python 构建你自己的轻量级流水线组件"
weight = 50
+++

轻量级 Python 组件不需要你为每次代码变更都构建一个新的容器镜像。它旨在在 notebook 环境中进行快速迭代。

相较于容器组件的优势：

* 快速迭代：不需要在每次更改后都构建新的容器镜像（构建镜像会花费时间）。
* 创作更轻松：组件可以在本地环境中创建。不需要 Docker 和 Kubernetes。

## 构建一个轻量级 Python 组件

要构建一个组件，首先要定义一个独立的 Python 函数，然后调用 `kfp.components.func_to_container_op(func)`
将该函数转换成一个可以在流水线中使用的组件。

组件函数需要满足以下几点要求：

* 函数必须是独立的。

  * 它不能包含任何在函数定义之外声明的代码。
  * 任何添加的导入都必须在主组件函数中。
  * 任何辅助函数也应在主组件函数中定义。

* 函数只能导入在基础镜像中可用的包。

  * 如果你需要导入一个在基础镜像中不可用的包那么你可以尝试换一个已经包含所需包的容器镜像。（另一种解决办法是你可以使用 
  `subprocess` 模块来运行 `pip install` 安装所需的包。）

* 如果函数进行运算，参数必须具有类型提示。支持的类型有 `int`，`float`，`bool`。所有其它类型的参数将作为字符串传递。
* 要构建一个能够输出多个值的组件，你需要使用 Python 的
  [typing.NamedTuple](https://docs.python.org/3/library/typing.html#typing.NamedTuple) 类型。

  语法提示：
  
    ```
    NamedTuple('MyFunctionOutputs', [('output_name_1', type), ('output_name_2', float)])
    ``` 
    
    `NamedTuple` 类已经导入，以便可以在函数声明中使用它。

## 教程

有关创建轻量级 Python 组件并在流水线中使用的示例，请参见 [轻量级 Python 组件基础](https://github.com/kubeflow/pipelines/blob/master/samples/core/lightweight_component/lightweight_component.ipynb) 中的 notebook。
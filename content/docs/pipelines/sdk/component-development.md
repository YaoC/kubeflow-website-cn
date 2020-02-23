+++
title = "创建可复用组件"
description = "有关创建可在各种流水线中使用的组件的详细教程"
weight = 40
+++

本页描述如何编写可复用的组件以便你可以将它作为流水线的一部分加载并使用。

如果你不了解流水线，查看 [流水线](/docs/pipelines/concepts/pipeline/)
和 [组件](/docs/pipelines/concepts/component/) 的概念指南。

本教程描述了手动编写完整的组件程序（使用任何语言）以及它的组件定义的方法。要通过 Python 函数快速构建组件请查看
[通过 Python 函数构建组件](/docs/pipelines/sdk/lightweight-python-components/) 和
[Python 组件中的数据传递](https://github.com/kubeflow/pipelines/blob/master/samples/tutorials/Data%20passing%20in%20python%20components.ipynb)。

## 总结

下面是创建和使用组件涉及的步骤的总结：

1.  编写包含你的组件逻辑的程序。该程序必须使用文件和命令行参数来与组件传递参数。
1.  将程序容器化。
1.  编写 YAML 格式的组件规范来描述 Kubeflow Pipelines 系统中的组件。
1.  使用 Kubeflow Pipelines SDK 来加载组件，在流水线中使用它并运行该流水线。

本页面余下部分给出关于输入和输出数据的解释，接着是上述步骤的详细描述。

## 组件和数据传递

组件的概念和函数的概念非常相似。每个组件可以有输入和输出（必须在组件规范中声明）。组件代码从输入中读取数据并为输出生产的数据。
接着流水线根据数据从某些组件的输出传递到某些组件的输入这一关系将这些组件实例连接起来。这和函数调用其它函数并在它们之间传递结果非常相似。
流水线系统处理实际的数据传递而组件负责消费输入的数据并生产输出数据。

## 往返于容器化程序的数据传递

创建组件时你需要思考组件如何与上游和下游的组件通信。也就是说，它如何消费输入数据和生产输出数据。

### 生产数据

要生产任何的数据，组件程序都必须将输出数据写到某个位置并告知系统该位置以便系统可以在步骤间传递数据。
程序必须接受将输出数据的位置作为命令行参数。也就是说，你不能硬编码路径。

#### 进阶：在外部系统中生产数据

在某些场景下，组件的目的是在某些外部服务中创建数据对象（例如 BigQuery 表）。在这种情况下，组件程序在完成工作后应该输出生产的数据的位置标识（例如
BigQuery 表名）而不是数据本身。该场景应该被限制在数据必须被放置在外部系统的情况下而不是在 Kubeflow Pipelines 系统中。组件的类别通常是
exporter（例如“上传数据到 GCS”）。注意流水线系统不能为超出其控制范围的数据提供一致性和可复现能力保障。

### 消费数据

命令行程序主要有两种常见的数据消费方式：

* 少部分的数据通常通过命令行参数传递：`program.py --param1 100`。
* 大量的数据或二进制数据通常存储在文件中然后将文件的路径传递给程序：`program.py --data1 /inputs/data1.txt`。
  系统需要意识到将数据放入文件中并将它们的路径传递给程序的需求。

## 编写程序代码

本节描述一个有两个输入（小规模数据和大规模数据）和一个输出的示例程序。示例中的编程语言是 Python 3。

### program.py

```python
#!/usr/bin/env python3
import argparse
from pathlib import Path

# 完成实际工作的函数（输出一个文本文件的前 N 行）
def do_work(input1_file, output1_file, param1):
  for x in range(param1):
    line = next(input1_file)
    if not line:
      break
    _ = output1_file.write(line)

# 定义并编译命令行参数
parser = argparse.ArgumentParser(description='My program description')
parser.add_argument('--input1-path', type=str, help='Path of the local file containing the Input 1 data.') # 路径应该被传入，而不是硬编码
parser.add_argument('--param1', type=int, default=100, help='Parameter 1.')
parser.add_argument('--output1-path', type=str, help='Path of the local file where the Output 1 data should be written.') # 路径应该被传入，而不是硬编码
args = parser.parse_args()

# 创建输出文件所在的目录（该目录可能存在也可能不存在）。
Path(args.output1_path).parent.mkdir(parents=True, exist_ok=True)

with open(args.input1_path, 'r') as input1_file:
    with open(args.output1_path, 'w') as output1_file:
        do_work(input1_file, output1_file, args.param1)
```

调用该程序的命令行是：

```
python3 program.py --input1-path <local file path to Input 1 data> \
                   --param1 <value of Param1 input> \
                   --output1-path <local file path for the Output 1 data>
```

## 编写 Dockerfile 来容器化你的应用

你需要一个 [Docker](https://docs.docker.com/get-started/) 容器镜像来打包你的程序。

创建容器镜像的指南并非是 Kubeflow Pipelines 特定的。为简单起见，本节提供的是创建标准容器的指南。
你可以使用任意方法来创建 Docker 容器。

你的 [Dockerfile](https://docs.docker.com/engine/reference/builder/)
必须包含所有的程序代码，包括包装程序和依赖（操作系统库、Python 包等）。 

确保你有一个镜像仓库的写权限以便你可以上传你的容器镜像。示例包括
[Google Container Registry](https://cloud.google.com/container-registry/docs/)
和 [Docker Hub](https://hub.docker.com/)。

给你的容器镜像去一个名称。本指南中使用的名称是 `gcr.io/my-org/my-image`。

### Dockerfile 示例

```
FROM python:3.7
RUN python3 -m pip install keras
COPY ./src /pipelines/component/src
```

创建一个 `build_image.sh` 脚本（见下面的示例）来根据 Dockerfile 构建容器镜像并将镜像上传到某个镜像仓库中。

运行 `build_image.sh` 脚本来根据 Dockerfile 构建容器镜像并将它上传到所选的镜像仓库中。

最佳实践：上传完镜像后，获取镜像的包含摘要的严格名称，并使用严格名称来进行复现。

### build_image.sh 示例：

```bash
#!/bin/bash -e
image_name=gcr.io/my-org/my-image # 在这里指定镜像名称
image_tag=latest
full_image_name=${image_name}:${image_tag}

cd "$(dirname "$0")" 
docker build -t "${full_image_name}" .
docker push "$full_image_name"

# 输出镜像的严格名称（包含来镜像的 sha256 摘要值）
docker inspect --format="{{index .RepoDigests 0}}" "${IMAGE_NAME}"
```

让你的脚本可执行：

```
chmod +x build_image.sh
```

## 编写组件定义文件

要用你的容器化程序创建组件你需要编写描述组件在 Kubeflow Pipelines 系统中的 YAML 格式的组件规范。

有关 Kubeflow Pipelines 组件的完整定义，参见 [组件规范](/docs/pipelines/reference/component-spec/)。
不过，对于本教程你无需了解组件规范的全部内容。本教程提供了足够的组件相关的信息。

通过在组件的实现一节中指定容器镜像来开始编写组件定义（`component.yaml`）：

```
implementation:
  container:
    image: gcr.io/my-org/my-image@sha256:a172..752f # 你推到镜像仓库中的容器镜像的名称
```

基于你的程序来完成组件实现一节：
 
```
implementation:
  container:
    image: gcr.io/my-org/my-image@sha256:a172..752f
    # 命令是一个字符串（命令行参数）列表。
    # 你可以使用 YAML 语言中两种列表语法中的任意一种。
    # 这里使用的是“流式”语法 —— 逗号分割的字符串放在方括号中。
    command: [
      python3, /kfp/component/src/program.py, # Path of the program inside the container
      --input1-path, <local file path for the Input 1 data>,
      --param1, <value of Param1 input>,
      --output1-path, <local file path for the Output 1 data>,
    ]
```

`command` 一节依然包含了一些虚设的占位符（在尖括号中）。*占位符* 表示一个在程序执行前会被替换成某个值或路径的命令行参数。在 `component.yaml`
中，你可以使用 YAML 的映射语法指定占位符从而和逐字字符串区别开。有三种占位符可用：

*   `{inputValue: 某个输入值}`   
    该占位符会被替换为指定输入的参数 **值** 。对于少量输入数据很有用。
*   `{inputPath: 某个输入值}`   
    该占位符会被替换成自动生成的 **本地路径** ，系统会在流水线运行时将输入数据传到组件中。
    该占位符指示系统将输入参数的数据写入到一个文件中，然后将该数据文件的路径传递到组件程序中。
*   `{outputPath: 某个输出值}`   
    该占位符会被替换成自动生成的 **本地路径** ，程序应该将输出数据写入路径对应的文件中。它会指示系统读取文件中的内容然后将它作为特定的输出值存储。

除了将真正的占位符放在命令行中之外，你需要在输入和输出一节中添加相应的输入和输出规范。输入/输出规范包含了输入名称、类型、描述和缺省值。仅有名称是必需的。
输入和输出名称是任意形式的字符串，但是需要注意 YAML 语法并在必要的时候使用引号。输入/输出名称不必与通常很短的命令行标志相同。
   
替换占位符如下所示：

+   将 `<local file path for the Input 1 data>` 替换成 `{inputPath: Input 1}` 并在 `inputs` 一节中添加 `Input 1`。
    字符串很小，所以作为命令行参数传递。
+   将 `<value of Param1 input>` 替换成 `{inputValue: Parameter 1}` 并在 `inputs` 一节中添加 `Parameter 1`。
    整型数值很小，所以作为命令行参数传递。
+   将 `<local file path for the Output 1 data>` 替换成 `{outputPath: Output 1}`
    并在 `outputs` 一节中添加 `Output 1`。 

替换完占位符并添加输入/输出后，你的 `component.yaml` 会是这样：

```
inputs: # 输入规范列表。每一个输入规范是一个映射。
- {name: Input 1}
- {name: Parameter 1}
outputs:
- {name: Output 1}
implementation:
  container:
    image: gcr.io/my-org/my-image@sha256:a172..752f
    command: [
      python3, /pipelines/component/src/program.py,

      --input1-path,
      {inputPath: Input 1}, # 引用自输入中的 "Input 1"

      --param1,
      {inputValue: Parameter 1}, # 引用自输入中的 "Parameter 1"

      --output1-path,
      {outputPath: Output 1}, # 引用自输出中的 "Output 1"
    ]
```

上面的组件规范是充分的，但是你需要添加更多的元数据来使它更有用。下面的示例包含了如下补充：

* 组件名称和描述。
* 针对每一个输入和输出：描述、缺省值和类型。
   
最终版本的 `component.yaml`：

```
name: Do dummy work
description: Performs some dummy work.
inputs:
- {name: Input 1, type: GCSPath, description='Data for Input 1'}
- {name: Parameter 1, type: Integer, default='100', description='Parameter 1 description'} # 缺省值必须指定为 YAML 字符串。
outputs:
- {name: Output 1, description='Output 1 data'}
implementation:
  container:
    image: gcr.io/my-org/my-image@sha256:a172..752f
    command: [
      python3, /pipelines/component/src/program.py,
      --input1-path,  {inputPath:  Input 1},
      --param1,       {inputValue: Parameter 1},
      --output1-path, {outputPath: Output 1},
    ]
```

## 通过 Kubeflow Pipelines SDK 将组件构建到流水线中

这是一个显示了如何加载组件并使用它来组成一个流水线的示例流水线。、
 
```python
import kfp
# 通过调用 load_component_from_file 或 load_component_from_url 加载组件
# 要加载组件，流水线作者只需要有访问 component.yaml 文件的权限。
# Kubernetes 集群执行流水线需要有访问组件中指定的容器镜像的权限。
dummy_op = kfp.components.load_component_from_file(os.path.join(component_root, 'component.yaml')) 
# dummy_op = kfp.components.load_component_from_url('http://....../component.yaml')

# 加载另外两个用于导入和导出数据的组件：
download_from_gcs_op = kfp.components.load_component_from_url('http://....../component.yaml')
upload_to_gcs_op = kfp.components.load_component_from_url('http://....../component.yaml')

# dummy_op 现在是一个接受组件输入的参数并生成一个任务对象（例如 ContainerOp 实例）的 “工厂函数”。
# 在 Jupyter Notebook 中通过输入 "dummy_op(" 后按 Shift+Tab 键来检查 dummy_op 函数
# 你还可以通过输入 help(dummy_op) 或 dummy_op? 或 dummy_op?? 来获取帮助
# dummy_op 函数的签名对应了组件的输入一节。
# 执行一些调整以使签名有效且符合 Python 规范：
# 1) 所有有缺省值的输入跟在没有缺省值的输入之后
# 2) 输入名称被转换成符合 Python 规范的名称（空格和符号替换成下划线并且字母小写）。

# 定义一个流水线并从组件创建任务：
def my_pipeline():
    dummy1_task = dummy_op(
        # 输入名称 "Input 1" 被转换成符合 Python 规范的参数名称 "input_1"
        input_1="one\ntwo\nthree\nfour\nfive\nsix\nseven\neight\nnine\nten",
        parameter_1='5',
    )
    # dummy1_task 的输出可以使用 dummy1_task.outputs 字典引用：dummy1_task.outputs['output_1']
    # ! 输出名称被转换成符合 Python 规范（”蛇形命名法“）的名称。

# 本流水线可以被编译、上传和提交以执行。
kfp.Client().create_run_from_pipeline_func(my_pipeline, arguuments={})
```

## 组织组件文件

本节提供了一种组织组件文件的推荐方法。你并不是一定要按这种方式组织文件。然后，使用标准的组织方式可以复用相同的脚本来进行测试、镜像构建和组件的版本控制。
有关真实的组件示例，参见
[示例组件](https://github.com/kubeflow/pipelines/tree/master/components/sample/keras/train_classifier)。

```
components/<component group>/<component name>/

    src/*            # 组件源码文件
    tests/*          # 单元测试
    run_tests.sh     # 运行测试的脚本
    README.md        # 文档。如果需要多个文件则移到 docs/ 目录中。

    Dockerfile       # 构建组件容器镜像的 Dockerfile
    build_image.sh   # 运行  docker build 和 docker push 的脚本

    component.yaml   # YAML 格式的组件定义
```

## 下一步

* 通过阅读有关设计和编写组件的 [最佳实践](/docs/pipelines/sdk/best-practices) 来巩固所学知识。
* 从 Python 函数直接 [构建轻量级组件](/docs/pipelines/sdk/lightweight-python-components/) 以便快速迭代。
* 查看如何 [从流水线导出指标](/docs/pipelines/metrics/pipelines-metrics/)。
* 通过 [为输出查看器添加元数据](/docs/pipelines/metrics/output-viewer/)
  来可视化你的组件输出。
* 浏览 [可复用组件和其它共享资源](/docs/examples/shared-resources/)。

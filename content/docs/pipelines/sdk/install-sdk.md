+++
title = "安装 Kubeflow Pipelines SDK"
description = "配置你的 Kubeflow Pipelines 开发环境"
weight = 20
+++

本指南告诉你如何安装用来构建机器学习流水线的
[Kubeflow Pipelines SDK](https://github.com/kubeflow/pipelines/tree/master/sdk)。你可以使用 SDK 来执行流水线，或者你也可以将流水线上传到
Kubeflow Pipelines UI 中执行。

SDK 中的所有的类和方法都在自动生成的 [SDK 参考文档](https://kubeflow-pipelines.readthedocs.io/en/latest/) 中有描述。

## 配置 Python

使用 Kubeflow Pipelines SDK 你需要 **Python 3.5** 或更高的版本。本指南使用 Python 3.7。

如果你还没有配置 Python 3 环境，那么立刻开始。本指南推荐使用 [Miniconda](https://conda.io/miniconda.html)，
但你也可以使用自己选择的虚拟环境管理器，例如 `virtualenv`。

按照以下步骤使用 [Miniconda](https://conda.io/miniconda.html) 配置 Python：

1. 根据你的环境选择下列方法之一安装 Miniconda：

  * Debian/Ubuntu/[Cloud Shell](https://console.cloud.google.com/cloudshell)：  

        ```bash
        apt-get update; apt-get install -y wget bzip2
        wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
        bash Miniconda3-latest-Linux-x86_64.sh
        ```

  * Windows：下载
    [安装包](https://repo.continuum.io/miniconda/Miniconda3-latest-Windows-x86_64.exe)
    并在安装过程中确保勾选了
    **Add Miniconda to my PATH environment variable** 选项。

  * MacOS：下载
    [安装包](https://repo.continuum.io/miniconda/Miniconda3-latest-MacOSX-x86_64.sh)
    后执行以下命令：

        ```bash
        bash Miniconda3-latest-MacOSX-x86_64.sh
        ```

1. 确认 `conda` 命令可用：

    ```bash
    which conda
    ```

    如果没有找到 `conda` 命令，那么将 Miniconda 添加到你的 path 中：
 
    ```bash
    export PATH=<YOUR_MINICONDA_PATH>/bin:$PATH
    ```

1. 使用你选择的名称创建一个干净的 Python 3 环境。本示例使用 Python 3.7，环境名称是 `mlpipeline`：
 
    ```bash
    conda create --name mlpipeline python=3.7
    conda activate mlpipeline
    ```
 
## 安装 Kubeflow Pipelines SDK

运行以下命令来安装 Kubeflow Pipelines SDK：

```bash
pip install https://storage.googleapis.com/ml-pipeline/release/latest/kfp.tar.gz --upgrade
```

安装成功后，`dsl-compile` 应该是可用的。你可以使用以下命令来进行验证：

```bash
which dsl-compile
```

该命令的响应应该和以下结果类似：

```
/<PATH_TO_YOUR_USER_BIN>/miniconda3/envs/mlpipeline/bin/dsl-compile
```

## 下一步

* [查看如何使用 SDK](/docs/pipelines/sdk/sdk-overview/)。
* [构建组件和流水线](/docs/pipelines/sdk/build-component/)。
* [开始使用 UI](/docs/pipelines/pipelines-quickstart)。
* [理解流水线相关概念](/docs/pipelines/concepts/)。

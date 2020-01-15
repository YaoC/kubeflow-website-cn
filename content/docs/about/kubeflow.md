+++
title = "Kubeflow"
description = "Kubeflow 介绍"
weight = 1
aliases = ["/docs/", "/docs/about/", "/docs/kubeflow/"]
+++

Kubeflow 项目致力于让机器学习（ML）工作流在 Kubernetes 上的部署更简单、轻便和可扩展。
我们的目标不是重新创建其他服务，而是提供一种在不同的基础设施下都能直接部署机器学习各领域最佳开源系统的方式。
无论在什么地方，只要你能运行 Kubernetes，就能运行 Kubeflow。

## 开始使用 Kubeflow

阅读 [Kubeflow 概览](/docs/started/kubeflow-overview/) 以了解 Kubeflow 的架构及如何使用 Kubeflow 去管理你的 ML 工作流。

跟随 [开始指南](/docs/started/getting-started/) 去设置你的环境并安装 Kubeflow。

## 什么是 Kubeflow?

Kubeflow 是 Kubernetes 的机器学习工具包。

要使用 Kubeflow，基本的流程如下：

* 下载并运行 Kubeflow 二进制安装文件
* 自定义配置文件
* 运行指定的脚本去部署你的容器到你特定的环境中

你可以调整配置以选择你想要在 ML 工作流的各个阶段（如：数据准备、模型训练、模型预测以及服务管理）使用的平台和服务。

你也可以选择将你的 Kubernetes 工作负载部署在本地、私有环境或云端环境。

阅读 [Kubeflow 概览](/docs/started/kubeflow-overview/) 以了解更多细节。

## Kubeflow 的使命

我们的目标是让机器学习模型的伸缩扩展及部署到生产环境尽可能地简单，主要通过让 Kubernetes 做它所擅长的事情来实现：

  * 可以简单、可重复及便携式地部署在不同的基础设施上（比如：在笔记本上实验，然后迁移到私有集群或公有云上）
  * 可以部署和管理松耦合的微服务
  * 可以按需伸缩扩展

因为机器学习从业者们要使用不同组合的工具，因此我们主要的目标之一是可以根据使用者的合理需求来自定义工具栈，并且让系统来处理那些繁杂琐碎的事务。
虽然我们从有限的一些工具开始，但我们也在与很多不同的项目一起工作，从而可以包含更多额外的工具。

最终，我们希望提供一组简单的清单，它能让你在已经运行 Kubernetes 的地方很简单的使用机器学习技术栈，并且可以基于部署的集群自己去调整配置。

## 历史

Kubeflow 始于 Google 将其内部运行 [TensorFlow](https://www.tensorflow.org/) 的方式开源出来，基于一个叫 [TensorFlow Extended](https://www.tensorflow.org/tfx/) 的流水线。
它开始只是一个将 TensorFlow jobs 运行在 Kubernetes 上的一种较简单的方式，但现在已经发展成为一个能运行整个机器学习流水线的多架构、多云框架。

## 参与贡献

有很多种方式可以为 Kubeflow 项目做贡献，我们十分欢迎大家做贡献！
阅读 [贡献者指南](/docs/about/contributing) 开始从源码着手贡献，通过 [社区指南](/docs/about/community) 来了解 Kubeflow 社区。

+++
title = "在 Azure AKS 上解决部署故障"
description = "帮助诊断及解决你可能在你的 Kubeflow 部署上遇到的问题"
weight = 100
+++

### 当访问 notebook 时，Jupyter Notebook 报错：'is not a valid page'

重启 ambassador 的 pod 一般都能解决该问题：`kubectl delete pods -l service=ambassador`

### The client does not have authorization to perform action...

这个可能是在确认你的订阅是否正确设置时引发的问题。要解决该问题，列出你的订阅列表然后改变活跃的订阅。

```
# 列出订阅列表
az account list --output table 

# 改变活跃的订阅
az account set --subscription "My Demos"
```
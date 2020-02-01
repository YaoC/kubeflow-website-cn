+++
title = "卸载 Kubeflow"
description = "卸载 Kubeflow 指南"
weight = 10
+++

在你的 Azure AKS 集群中卸载 Kubeflow

```
# 进入你的 Kubeflow 部署目录
cd ${KF_DIR}

# 移除 Kubeflow
kfctl delete -f ${CONFIG_FILE}
```
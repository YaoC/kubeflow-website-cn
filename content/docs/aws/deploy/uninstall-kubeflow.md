+++
title = "卸载 Kubeflow"
description = "卸载 Kuberflow 操作指南"
weight = 10
+++


## 卸载 Kubeflow

```
cd ${KF_DIR}
kfctl delete -f ${CONFIG_FILE}
```

> 注意：如果你将 Kubeflow 安装在一个已存在的亚马逊 EKS 集群上，上述脚本不会在这步卸载你的集群。如果你想要关闭 EKS 集群，你必须自己手动删除它。

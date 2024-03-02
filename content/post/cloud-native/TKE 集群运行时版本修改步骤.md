+++
author = "Jokey Li"
title = "TKE 集群运行时版本修改步骤"
date = "2021-05-20"
description = "TKE 集群运行时版本修改步骤"
featured = true
tags = [
    "TKE",
]
categories = [
    "操作教程",
]
series = ["云原生"]
thumbnail = "images/materials/jokey_smurf_sa.png"
+++


# 运行时版本修改

登录TKE 控制台，点击【基本信息】->【运行时组件】编辑按钮修改运行时版本，如下图：

![修改运行时版本](/images/articles/xb56ck3vct.png)

修改完毕后，集群中新添加节点将使用新的版本运行时。

# 存量节点修改

**⚠️ 存量节点修改运行时建议将节点【移出】再【添加已有节点】加回集群的方式，此方式会重新安装系统并初始化节点，请自行评估风险和操作时间点。**

### 操作步骤

先点击【封锁】节点，目的是不让新的 Pod 再调度到要操作的节点上：

![第一步：封锁节点](/images/articles/am86tir8jl.png)

再点击【驱逐】节点 Pods，目的是将节点上的存量 Pods 优雅迁出：

![第二步：驱逐节点 Pods](/images/articles/0dqfwhxuj5.png)

待驱逐 Pods 过程完成后，点击移出节点并取消勾选“销毁节点”，这样可以不销毁按量付费的 CVM：

![第三步：移出节点](/images/articles/e4dai50yyx.png)

最后一步，重新将迁出的节点加入集群：

![第四步：重新加入集群](/images/articles/w2c38ri13n.png)

![选择添加之前迁出的节点](/images/articles/fgmdof1les.png)

待节点添加进集群完成后，该示例节点将使用修改后的运行时版本，操作完后记得检查节点的运行时版本和状态是否正常。
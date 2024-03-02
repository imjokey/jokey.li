
+++
author = "Jokey Li"
title = "TKE 容器绑定 EIP 后使用 EIP 网络访问节点外"
date = "2022-05-11"
description = "TKE 容器绑定 EIP 后使用 EIP 网络访问节点外"
featured = true
tags = [
    "kubernetes",
    "TKE",
    "NAT"
]
categories = [
    "操作教程",
]
series = ["云原生"]
thumbnail = "/images/materials/cola.jpeg"
+++


# 问题

在 TKE 中配置 [ Pod 直接绑定弹性公网 IP 使用](https://cloud.tencent.com/document/product/457/64886) EIP 功能后， 可以通过绑定的 EIP 直接访问 Pod ， 但是从 Pod 中访问节点外的网络时还是走的节点网络，而不是绑定的 EIP （网络）。

# 原因

这是因为从 Pod 访问节点外网络（以公网为例）时， Pod 网段出节点会被做 SNAT 策略导致， 相关说明参考：[容器访问节点外服务时是否做 SNAT 配置](https://cloud.tencent.com/developer/article/1812833)。

# 解决办法

**需要配置下访问不做SNAT 就行了，有两种配置策略**：

1. 配置不做 SNAT 要访问的目的网段, 参考配置：[容器访问节点外服务时是否做 SNAT 配置](post/cloud-native/访问节点外服务时是否做-snat-配置/)。
2. 配置不做 SNAT的 源 IP（同样是修改 ip-masq-agent 配置）。

> 配置条件：当前集群 ip-masq-agent 镜像版本 v2.6.1 及以上

增加不做 SNAT 的源网段（以 10.0.0.0/16 为例）配置：

```js
kubectl edit cm -n kube-system ip-masq-agent-config 
```

将下列数据添加到 data.config 中：

```js
NonMasqueradeSrcCIDRs":["10.0.0.0/16"]
```

如图所示：

![配置示例](/images/articles/4dqbz69tab.png)

等待 "ResyncInterval" 时间周期（默认1分钟）后测试看看配置是否生效。

​
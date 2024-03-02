+++
author = "Jokey Li"
title = "TKE 集群容器访问节点外服务时是否做 SNAT 配置"
date = "2021-09-23"
description = "TKE 集群容器访问节点外服务时是否做 SNAT 配置"
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

## 适用的场景

TKE 中无论是GR模式还是VPC-CNI 模式都是基于路由的 underlay 网络，并非 overlay，所以理论上容器访问节点外可以做SNAT，也可以不用做SNAT。

但在 TKE 中无论是 Global Router 还是 VPC-CNI 网络模式，在容器内访问集群所在 VPC 网段和容器网段默认是不会做 SNAT 的，除此之外访问其他网段都是会做 SNAT 的，当某些业务场景下需要保留容器源 IP 时，我们就需要修改相关配置来避免访问某些 IP 或网段时做 SNAT，从而实现保留容器源 IP 的需求。
## 操作步骤

在可以使用 kubectl 连接到集群的环境中，**执行下面命令在资源的"NonMasqueradeCIDRs" 字段列表中添加不想做 SNAT 访问的目的 IP 或网段。相应的，如果想让访问特定网段时做 SNAT，将特定网段从列表中删除即可：**

```bash
kubectl edit cm  ip-masq-agent-config -n kube-system
```

修改说明如下图所示（注意 YAML 格式）：

![操作示例](/images/articles/3l3earx2jj.png)

等待 "ResyncInterval" 时间周期（默认1分钟）后测试看看配置是否生效。
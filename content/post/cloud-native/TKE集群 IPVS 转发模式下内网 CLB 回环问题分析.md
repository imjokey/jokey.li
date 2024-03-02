+++
author = "Jokey Li"
title = "TKE集群 IPVS 转发模式下内网 CLB 回环问题分析"
date = "2021-05-11"
description = "TKE集群 IPVS 转发模式下内网 CLB 回环问题分析"
featured = true
tags = [
    "kubernetes",
    "NAT"
]
categories = [
    "问题排查",
]
series = ["云原生"]
thumbnail = "images/materials/lickdog.jpeg"
+++

# **问题描述**

有客户反馈集群中两个 Service 之间调用有偶发超时现象，经过排查后发现是触发了 TKE 中的内网 CLB 回环问题导致（相同场景下公网CLB 无此回环问题 ），但客户又反馈另一个集群也有类似的调用场景，但一直没有出现过超时现象。经过查看对比，两个集群中 Service 之间调用场景确实是一致的，但两个集群中被调用服务的  Service 中 `externalTrafficPolicy` 配置有差异，有回环问题的集群配置为"Local"，无回环问题的集群配置为 "Cluster"。

# **先说结论**

**为啥使用 "externalTrafficPolicy=Local "有回环问题，而使用 "externalTrafficPolicy=Cluster" 无回环问题？**

分别在两个不同集群的出节点网卡抓包发现，有回环问题的集群在 Pod A 访问 CLB IP 出节点时没有做 SNAT，没有回环问题的集群访问时出节点做了 SNAT（请求源 IP 转换成节点 IP），又由于 CLB 本身转发时会判断请求源 IP，如果发现转发的后端列表中有和请求源相同 IP 的后端，就不会转发给这个后端，⽽选择其它后端，所以做了 SNAT 的集群刚好避免了回环问题。

​

# **问题分析**

**触发回环问题的场景：**

当一个集群中的容器Pod A 中调用通过内网 CLB 暴露的 Service B 服务（Pod B）时可能会发生。

**回环问题触发的链路**：

PodA（client） -> CLB(内网) -> Service B（刚好转发到 Pod A 节点的 NodePort ）-> Pod 。

由于两个集群的部署场景是一致的，也就是满足回环问题的触发场景，**在 TKE IPVS 转发模式下, pod 内访问负载均衡器类型的服务报文是需要出节点的（因为 LB IP 没有绑定在 ipvs0 接口），如此根据默认 iptables 规则出节点应该是要做 SNAT的**， 但是有回环问题到集群实际上出节点实际上并没有做 SNAT， 所以接下来分析一下 Service 中 `externalTrafficPolicy` 的不同配置对于容器网络包的转发链路影响。

#### **对比 iptables 规则差异**

首先对比两个集群节点中 iptables（NAT 表）的转发规则情况：
![iptables 规则差异](/images/articles/r0avrt81k2.png)

发现集群中 Service 设置了  “externalTrafficPolicy=Local ” 的节点会多出两条专门为 "externalTrafficPolicy=Local"添加的 的 iptables 规则。

#### **规则差异如何影响容器出节点包的转发：**

externalTrafficPolicy 配置为 Local  本身的作用是保留客户端源 IP 并避免 LoadBalancer 和 NodePort 类型服务的第二跳转发， 详情参考：[externalTrafficPolicy 介绍](https://kubernetes.io/zh/docs/tasks/access-application-cluster/create-external-load-balancer/#%E4%BF%9D%E7%95%99%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%BA%90-ip)。

根据 iptables 规则知识，我们看到当网络包要出节点时，都要先命中`OUTPUT`链：

```bash
-A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
```

然后根据后续链转发规则一路往下走看看：

Service 使用默认  “externalTrafficPolicy=Cluster ” 的集群节点，做了SNAT 的情况：
![出节点 SNAT 分析](/images/articles/fpb9wb8ddo.png)

Service 设置了  “externalTrafficPolicy=Local ” 的集群节点，没有做 SNAT 的情况：
![出节点没有 SNAT 分析](/images/articles/mcsf0gkaum.png)

从上面对比可以看出， 虽然 Service 的 externalTrafficPolicy 配置本来是为了处理访问 Service  入方向流量，但在添加规则时确实影响了容器访问 CLB 出节点时的策略，从而决定了是否具有内网 CLB 回环问题。

​

参考资料：https://kubernetes.io/blog/2018/07/09/ipvs-based-in-cluster-load-balancing-deep-dive/
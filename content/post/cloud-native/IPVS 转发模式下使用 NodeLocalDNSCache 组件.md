+++
author = "Jokey Li"
title = "在 IPVS 转发模式下使用 NodeLocalDNSCache 组件"
date = "2021-09-23"
description = "IPVS 转发模式下使用 NodeLocalDNSCache 组件"
featured = true
tags = [
    "kubernetes",
    "TKE"
]
categories = [
    "操作教程",
]
series = ["云原生"]
thumbnail = "images/materials/fox.jpeg"
+++

# 使用场景

在集群中以 Daemonset 的方式运行 [NodeLocal DNS Cache](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns?spm=a2c6h.12873639.0.0.b8e3669eIhJqEN) 组件，能够大幅提升集群内 DNS 解析性能，以及有效避免 [conntrack 冲突引发的 DNS 五秒延迟](https://www.weave.works/blog/racy-conntrack-and-dns-lookup-timeouts)。

目前 TKE 已经将 NodeLocal DNS Cache 作为增强组件供用户在集群中安装使用，但是目前仅限 Kube-proxy 转发模式为 Iptables 的集群安装，下面将介绍下在转发模式为 IPVS  如何部署使用 NodeLocal DNS Cache 。

# 操作步骤

## 1. 存量节点安装 

1. 根据社区示例的 [nodelocaldns.yaml](https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/nodelocaldns/nodelocaldns.yaml) 准备一个资源清单，把它保存为`nodelocaldns.yaml`。

2. 把`nodelocaldns.yaml`清单里的变量更改为正确的值：

```bash
DNS_SERVICE="kube-dns"
DNS_CLUSTER_IP=`kubectl get svc ${DNS_SERVICE} -n kube-system -o jsonpath={.spec.clusterIP}`
CUSTOM_DOMAIN="cluster.local"
NODE_LOCAL_DNS="169.254.20.10"
sed -i "s/__PILLAR__LOCAL__DNS__/$NODE_LOCAL_DNS/g; s/__PILLAR__DNS__DOMAIN__/$CUSTOM_DOMAIN/g; s/,__PILLAR__DNS__SERVER__//g; s/__PILLAR__CLUSTER__DNS__/$DNS_CLUSTER_IP/g" nodelocaldns.yaml
```

替换的变量名说明：

- `DNS_CLUSTER_IP`：可通过在集群中执行`kubectl get svc <DNS_SERVICE>-n kube-system -o jsonpath={.spec.clusterIP}`命令获取，其中 `<DNS_SERVICE>` 为集群 DNS 服务的 Service 名，在 TKE 集群中为 "kube-dns"。
- `CUSTOM_DOMAIN`：K8S集群创建时如果没有指定的话，默认值是 "cluster.local"。 
- `NODE_LOCAL_DNS`：是 NodeLocalDNSCache 在节点上监听的 IP 地址，推荐直接使用 "169.254.20.10" IP。

3. 应用部署 NodeLocal DNSCache 组件资源：

```bash
 kubectl create -f nodelocaldns.yaml
```

4. 修改 kubelet 参数：

由于 kube-proxy 运行在 IPVS 模式，需要修改 kubelet 的`--cluster-dns`参数为 NodeLocal DNSCache 在节点上监听的`NODE_LOCAL_DNS`地址，集群中所有节点依次执行以下命令，修改 kubelet 启动参数并重启。

```bash
NODE_LOCAL_DNS="169.254.20.10" 
sed -i "/CLUSTER_DNS/c\CLUSTER_DNS=\"--cluster-dns=${NODE_LOCAL_DNS}\"" /etc/kubernetes/kubelet
systemctl restart kubelet
```

> ⚠️ NodeLocalDNSCache 组件安装配置好后存量 Pods 还是使用的`DNS_CLUSTER_IP`解析，存量 Pods 需要重建或者修改配置`dnsConfig`后生效。

## 2. 新增节点配置

当存量节点已经部署运行了 [NodeLocal DNS Cache](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns?spm=a2c6h.12873639.0.0.b8e3669eIhJqEN) 组件时，新增节点只需要自定义配置 kubelet 参数`--cluster-dns`为上述`NODE_LOCAL_DNS`地址即可，目前自定义参数需要开白支持，可联系 TKE 售后同学帮忙开启。

## 3. 卸载组件资源（谨慎）

如果不想再使用此功能，卸载步骤如下：

1. 先恢复对 kubelet 配置所做的所有改动（注意变量名）。

```bash
DNS_CLUSTER_IP=${DNS_CLUSTER_IP}
sed -i "/CLUSTER_DNS/c\CLUSTER_DNS=\"--cluster-dns=${DNS_CLUSTER_IP}\"" /etc/kubernetes/kubelet
systemctl restart kubelet
```

2. 再删除部署的 NodeLocal DNS Cache 的所有资源：

```bash
kubectl delete -f nodelocaldns.yaml
```

> ⚠️ 对应的，已经有使用`NODE_LOCAL_DNS`解析的存量 Pods 需要重建或者修改配置`dnsConfig`后生效。

## 参考

[https://kubernetes.io/zh/docs/tasks/administer-cluster/nodelocaldns](https://kubernetes.io/zh/docs/tasks/administer-cluster/nodelocaldns)

[https://cloud.tencent.com/document/product/457/40613](https://cloud.tencent.com/document/product/457/40613)

​
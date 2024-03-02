
+++
author = "Jokey Li"
title = "修改 k8s 集群 CorenDNS 服务地址"
date = "2021-09-23"
description = "修改 k8s 集群 CorenDNS 服务地址"
featured = true
tags = [
    "kubernetes",
    "TKE"
]
categories = [
    "操作教程",
]
series = ["云原生"]
thumbnail = "images/materials/pig.jpeg"
+++

## **使用场景**

TKE 中的 Coredns 服务 IP 目前没有办法在创建集群的时候指定，默认是从客户配置的 Service CIDR 网段中分配一个随机的 IP,下面将介绍如何自定义修改 Coredns 服务 IP。

## **前提条件**

1. 在配置的 service CIDR 网段中选择一个目前集群中没有被使用的 IP
2. 集群中存量的 Pods 可以接受被重建。

## **操作步骤**

**1.修改 kubelet 启动参数**
先到存量节点上修改 kubelet 配置文件中 `--cluster-dns` 参数为新的 Cluster IP 并重启 kubelet，操作命令如下：

```bash
DNS_CLUSTER_IP=xxx.xxx.xxx.xxx
sed -i "/CLUSTER_DNS/c\CLUSTER_DNS=\"--cluster-dns=${DNS_CLUSTER_IP}\"" /etc/kubernetes/kubelet
systemctl restart kubelet
```

### 2.重建指定了新 ClusterIP 的 Coredns 的 Service 资源

由于`.spec.clusterIP` 字段是不可修改的，所以必须先删除原来的Service 资源：

```bash
kubectl delete svc kube-dns -n kube-system
```

然后再重新创建 Service kube-dns，可以根据需求适当修改和应用如下 YAML重建：

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/port: "9153"
    prometheus.io/scrape: "true"
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: CoreDNS
  name: kube-dns
  namespace: kube-system
spec:
  clusterIP: xxx.xxx.xxx.xxx  # 你要指定的服务 IP
  ports:
  - name: dns-tcp
    port: 53
    protocol: TCP
    targetPort: 53
  - name: dns
    port: 53
    protocol: UDP
    targetPort: 53
  selector:
    k8s-app: kube-dns
  sessionAffinity: None
  type: ClusterIP
```

### 3.重建集群中存量已有所有 Pods

由于kubelet 使用 --cluster-dns=<DNS 服务 IP> 标志将 DNS 解析器的信息传递给每个容器，所以存量 Pods 需要重建下更新到新的  Cluster IP，使用正常删除命令即可：

```bash
 kubectl delete pod --all --all-namespaces 
```

### 4.新建节点时指定 kubelet 参数

新建节点时可以通过自定义参数的功能指定kubelet 配置文件中`--cluster-dns`参数，但目前需要联系售后同学开白名单后可在控制台配置使用。


> 温馨提示：由于 Coredns Pods 比较特殊，在 Coredns 创建时容器中 /etc/resolv.conf 默认是继承当前节点相同路径中的Dnsservers 配置，不会使用 kubelet 参数中的配置，并且后续也不会 watch 和同步节点 /etc/resolv.conf  中的内容改动，所以节点上配置修改后需要重建 coredns Pods 更新新配置。
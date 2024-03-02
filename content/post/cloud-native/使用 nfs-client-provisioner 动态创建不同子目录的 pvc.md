
+++
author = "Jokey Li"
title = "使用 nfs-client-provisioner 动态创建不同子目录的 pvc"
date = "2021-07-08"
description = "使用 nfs-client-provisioner 动态创建不同子目录的 CFS 类型 PVC"
featured = true
tags = [
    "kubernetes",
    "TKE"
]
categories = [
    "操作教程",
]
series = ["云原生"]
thumbnail = "images/materials/rabbit.jpeg"
+++


# 使用场景

目前使用 StorageClass 自动创建 CFS 类型 PVC 和 PV，每个 PV 都需要对应一个文件系统（CFS 实例），如果想要多个 PV（不同子路径） 使用同一个文件系统，就需要手动创建 PV 时指定 CFS 文件系统的具体的路径然后绑定 PVC 使用，这是一种办法，但是当需要的 PV 数量多了就会非常繁琐， 对于此使用场景我们可以使用社区的 [nfs-client-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) 项目来实现动态创建 CFS 文件系统中的子路径，接下来我们来介绍下如何在 TKE 中使用`nfs-client-provisioner`。

# 操作步骤

## 1.准备 CFS 文件系统实例

 使用 [CFS-CSI 插件 ](https://cloud.tencent.com/document/product/457/44234#.E5.AE.89.E8.A3.85.E6.96.87.E4.BB.B6.E5.AD.98.E5.82.A8.E6.89.A9.E5.B1.95.E7.BB.84.E4.BB.B6)创建或通过 [CFS 控制台](https://console.cloud.tencent.com/cfs/) 创建一个 CFS 文件系统实例。

## 2.安装部署`nfs-subdir-external-provisioner`

官方提供两种安装方式， [helm 安装](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner#with-helm) 和 [手动部署 YAML 安装](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner#without-helm)，这里为了方便，我们采用 helm 安装的方式。

1.在访问集群的客户端安装 [helm](https://helm.sh/)

可以是在集群节点中，也可以是本地能连接集群的客户端，安装 helm3 参考 [helm 安装](https://helm.sh/zh/docs/intro/install/) 。

2.安装部署

可以根据需求修改指定参数后部署：

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
# 下载 helm chart 文件至本地目录，查看可以指定的 values 选项（可选）
helm pull nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --untar
# 使用类似下面命令安装 nfs-subdir-external-provisioner 资源
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=x.x.x.x \
    --set nfs.path=/exported/path \
    --set image.repository=xxxxx \
    --set image.tag=xxx
    ...
```

## 3 . 配置使用 CFS 文件系统子目录的 PVC 。

使用上一步部署的`nfs-subdir-external-provisioner`动态创建存储卷。

部署后会默认生成一个存储类资源，默认存储类名是"nfs-client"(也可以在部署时自定义指定)，如下：

```bash
[root@VM-0-85-tlinux ~]# kubectl  get sc
NAME            PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
...
nfs-client      cluster.local/nfs-subdir-external-provisioner   Delete          Immediate              true                   16d
...
```

然后使用上述生成的存储类动态创建存储卷：

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client 
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

等待 PVC 状态为 Bound ,则说明动态创建成功：

```bash
[root@VM-0-126-tlinux ~]# kubectl  get pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test-claim   Bound    pvc-a3751ae4-4937-4fdc-92d4-6706eec8e01c   1Mi        RWX            nfs-client     10s
```

在`nfs-subdir-external-provisioner` Pod 所在节点查看已经自动创建了对应 PVC 的子目录，如下图：

```bash
[root@VM-0-85-tlinux ~]# df -h | grep <nfs server>
<nfs server>:/      14G   32M   14G   1% /var/lib/kubelet/pods/d12c6680-0871-44db-87ab-2e0cd384649a/volumes/kubernetes.io~nfs/nfs-subdir-external-provisioner-root
[root@VM-0-85-tlinux ~]# ls /var/lib/kubelet/pods/d12c6680-0871-44db-87ab-2e0cd384649a/volumes/kubernetes.io~nfs/nfs-subdir-external-provisioner-root
default-test-claim-pvc-a3751ae4-4937-4fdc-92d4-6706eec8e01c
```

然后创建一个工作负载设置挂载上面生成的 PVC， 在 Pod 所在节点可以看到 PVC 挂载点说明可以正常使用：

```bash
[root@VM-0-126-tlinux ~]# df -h | grep <nfs server>
<nfs server>:/default-test-claim-pvc-a3751ae4-4937-4fdc-92d4-6706eec8e01c   14G   32M   14G   1% /var/lib/kubelet/pods/0cbf27d3-5357-4b9c-bda4-6e4c8cdc3b4a/volumes/kubernetes.io~nfs/pvc-a3751ae4-4937-4fdc-92d4-6706eec8e01c
[root@VM-0-126-tlinux ~]# ls /var/lib/kubelet/pods/0cbf27d3-5357-4b9c-bda4-6e4c8cdc3b4a/volumes/kubernetes.io~nfs/pvc-a3751ae4-4937-4fdc-92d4-6706eec8e01c
text.txt  # pod中产生的数据
```

# 总结

本文使用社区的 [nfs-client-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) 项目实现了在 TKE 集群只使用一个 CFS 文件系统实例，动态创建多个不同子路径的 PVC 供工作负载挂载。
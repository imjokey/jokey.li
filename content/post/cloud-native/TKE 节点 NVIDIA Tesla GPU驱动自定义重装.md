+++
author = "Jokey Li"
title = "TKE 节点 NVIDIA Tesla GPU驱动自定义重装"
date = "2021-06-01"
description = "TKE 节点 NVIDIA Tesla GPU驱动自定义重装操作步骤"
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

# 使用场景

默认情况下，用户在 TKE 添加 GPU 节点时，会自动预装特定版本 GPU 驱动，但是目前默认安装 GPU 驱动版本是固定的，用户还不能选择要安装的 GPU 驱动版本，当用户有其他版本的 GPU 驱动使用需求时，就需要在节点上重新安装，下面将介绍在 TKE 节点中如何重新安装 GPU 驱动程序。

# 操作步骤

## 1.卸载原驱动

先卸载原驱动，在节点上执行卸载命令：

```
nvidia-uninstall
```

原驱动卸载过程如下图所示：

![没有使用相关配置，所以选择不备份](/images/articles/uy4uuty8mr.png)

提示卸载原驱动完成即表示成功卸载：

![卸载完成](/images/articles/lamx9wgana.png)

## 2.重启节点

**由于驱动是被编译进内核加载的，卸载完原驱动需要重启下节点，不重启会因原驱动还在加载中导致安装新驱动失败。**

## 3.下载新驱动程序并安装

登录 [NVIDIA 驱动下载](http://www.nvidia.com/Download/Find.aspx) 官网下载选择 linux 64 bit shell 安装文件，如下图：

![下载新驱动安装文件](/images/articles/wgle6nspry.png)

这里我们选择安装 NVIDIA Tesla 10.2 版本驱动，最终可通过类似如下命令中的链接下载 shell 安装脚本到节点中并执行安装：

```
wget https://us.download.nvidia.com/tesla/440.95.01/NVIDIA-Linux-x86_64-440.95.01.run
chmod +x NVIDIA-Linux-x86_64-440.95.01.run
sh NVIDIA-Linux-x86_64-440.95.01.run
```

新驱动安装过程如下图：

![选择 YES](/images/articles/ufr7zbz4nj.png)

等待新驱动安装完成：

![安装完成](/images/articles/4lq6xe3jd4.png)

## 4.测试新驱动

- 在节点上执行`nvidia-smi`查看 GPU 情况，可查看到 GPU 信息并显示驱动版本为新版本：

![查看 GPU 信息](/images/articles/kne82evtd8.png)

- 查看 k8s 是否识别到节点 GPU 容量，执行命令：

```
kubectl describe node <NodeName>
```

 从 k8s 节点资源查看 GPU 资源是否和实际资源一致，如下图：

![k8s 节点资源](/images/articles/onzx8uih6p.png)

# 总结

本文简单介绍了如何在 TKE 重新安装 GPU 驱动程序，如有相关需求可按照上述操作安装。

​

参考资料：https://cloud.tencent.com/document/product/560/8048
+++
author = "Jokey Li"
title = "k8s 问题排错小技巧"
date = "2021-05-23"
description = "k8s 常用的排查命令梳理，不定期更新..."
featured = true
tags = [
    "kubernetes",
    "tcpdump"
]
categories = [
    "问题排查",
]
series = ["云原生"]
thumbnail = "images/materials/wolf.jpeg"
+++




# 1. 容器环境中没有 shell 环境

有时候我们想要查看下容器内部的一些东西，但是无奈容器没有shell 执行环境，比如想看看 coredns 容器中 `/etc/resolv.conf` 的内容是否正确继承了节点的配置，比较简单的操作步骤如下（以 docker 运行时为例）：

登录到容器所在节点上，执行 `docker ps` 查找到 coredns 容器 ID，然后再使用 `cp` 命令把文件复制出来就可以看了：

```bash
docker ps | grep <容器名> 
docker cp <容器ID>:/want/to/see/dir .
```

# 2. 常用排查命令

1.查看 kubelet 守护服务日志（运行时日志类似）

```bash
# 查找相关 daemon 服务
systemctl list-units | grep <DAEMON_SERVICE_NAME> 

# 显示 kubelet 滚动日志
journalctl  -u kubelet -f

# 导出 kubelet 日志到文件
journalctl  -u kubelet > k.log 

# 查看 kubelet 2 小时之前到现在的日志
journalctl  -u kubelet  --since "2 hours ago" | less
```

2.获取当前资源配置的 YAML 

```bash
kubectl get <RESOURCE> <POD_NAME> -n <NAMESPACE> -o yaml 

命令示例: 
kubectl get pod nginx-xxx -n default -o yaml 
kubectl get deploy nginx -n default -o yaml
```

3.查看资源当前状态描述 

```bash
kubectl describe <RESOURCE>  <POD_NAME> -n <NAMESPACE>

命令示例: 
kubectl describe  pod nginx-xxx -n default
kubectl describe  pvc nginx -n default
```

4.查看容器日志

```bash
# 动态刷新查看 Pod 中指定容器的后 20 行日志 
kubectl logs <POD_NAME> -c <CONTAINER_NAME> --tail 20 -f  -n <NAMESPACE>

命令示例: 
kubectl logs nginx-xxx -c nginx --tail 20 -f -n default 
```

5.打印出资源指定的字段值（YAML结构)

```bash
kubectl get <RESOURCE> -o custom-columns=<ALIAS_NAME_1>:<RESOURCE_KEY_1>,<ALIAS_NAME_2>:<RESOURCE_KEY_2>

命令示例: 
# 分别打印资源中的 .metadata.name（别名 Name）,.status.eniInfos（别名 eni） 字段值。
kubectl get nec -ocustom-columns=Name:.metadata.name,eni:.status.eniInfos
# 如果字段中有"."符号, 需要整体使用""扩起来再转译, 比如"tke.cloud.tencent.com/eni-ip" 写成 "tke\.cloud\.tencent\.com/eni-ip"
kubectl get no -o=custom-columns=NAME:.metadata.name,Allocatable_eni-ip:.status.allocatable."tke\.cloud\.tencent\.com/eni-ip"
```

6.使用 kubectl 执行容器命令

```bash
kubectl exec -it <POD_NAME> -c <CONTAINER_NAME> -n <NAMESPACE> -- <COMMAND>

命令示例: 
kubectl exec -it nginx-xxx -c nginx -n default  -- sh
kubectl exec -it nginx-xxx -c nginx -n default  -- sleep 100000
```

7.使用 kubectl 创建测试 Pod 

```bash
kubectl run busybox --image=busybox --overrides='{ "spec": { "nodeName": "<NODE_NAME>" } }' --command -- sleep 100000

命令示例：
# 测试 Pod 运行在指定的节点"10.0.5.3"上
kubectl run busybox --image=busybox --overrides='{ "spec": { "nodeName": "10.0.5.3" } }' --command -- sleep 100000
```

8.更多 kubectl 命令用法

```bash
kubectl 命令各种使用方法：
kubectl --help 
```

# 3. 常用抓包命令

```bash

# 查看容器ID
docker ps | grep <CONTAINER_NAME>

# 查看容器进程ID
docker inspect <CONTAINER_ID> | grep Pid

# 进入进程的网络命名空间
nsenter -t <TARGET_PID> -n

# tcpdump 抓包
tcpdump -i <INTERFACE_NAME> host <HOST> and port <PORT> -nve

命令示例:
# 查看某容器ID
docker ps | grep xxx

# OUTPUT: 
# ede761bade47  xxxx

# 查看某容器进程ID
docker inspect  ede761bade4 | grep Pid
# OUTPUT:
#            "Pid": 13062,
#            "PidMode": "",
#            "PidsLimit": null,

# 使用nsenter 进入进程的网络命名空间
nsenter -t 13062 -n

# 查看在 eth0 接口 ip 为8.8.8.8,端口号为 53 的非域名显示详细包
tcpdump -i eth0 host 8.8.8.8 and port 53 -nvve 

# 查看在 eth0 接口 src ip 为8.8.8.8, dst ip 为 1.1.1.1 的非域名显示详细包
tcpdump -i eth0 src 8.8.8.8 and dst 1.1.1.1 -nvve 

# 使用 Wireshark 分析完整报文文件 dns.pcap
tcpdump -i eth0 host 8.8.8.8 and port 53 -s 0 -w dns.pcap
```
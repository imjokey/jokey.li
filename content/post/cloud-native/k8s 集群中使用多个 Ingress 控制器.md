
+++
author = "Jokey Li"
title = "k8s 集群中使用多个 Ingress 控制器"
date = "2021-04-30"
description = "k8s 集群中使用多个 Ingress 控制器"
featured = true
tags = [
    "kubernetes",
    "ingress",
    "TKE"
]
categories = [
    "操作教程",
]
series = ["云原生"]
thumbnail = "images/materials/surprisedog.jpeg"
+++

# 背景

TKE 服务在集群内默认启用了基于腾讯云负载均衡器实现的 Ingress，支持 HTTP、HTTPS，同时也支持在集群内自建其他 Ingress 控制器，可以根据业务需要选择不同的 Ingress 类型，比如常用的 Nginx Ingress 控制器。

# 安装方式

方式一：通过 TKE 产品化的 Nginx ingress 组件安装，详情参考：[TKE Nginx Ingress](https://cloud.tencent.com/document/product/457/50502)

方式二：通过 TKE 控制台 【应用市场】 安装 Nginx Ingress ，应用市场安装说明参考：[TKE 应用市场 ](https://cloud.tencent.com/document/product/457/46432)

方式三：通过官网文档使用 helm 安装 Nginx Ingress，详情参考：[Helm 安装 Nginx Ingress ](https://kubernetes.github.io/ingress-nginx/deploy/#using-helm)

# 使用配置

下面将分别介绍在 TKE 中常用的两种 Ingress 类型的使用和多个 Ingress 控制器如何共同使用。

## 基于腾讯云 CLB 的 Ingress

TKE 默认提供基于腾讯云 CLB 的 Ingress 功能，用户可以直接在控制台【服务与路由】的 【Ingress】中根据需要暴露七层服务，也可以通过应用 Ingress YAML 资源来创建配置，基于 CLB 的 Ingress 控制器管理逻辑如下：  

- 当 Ingress 资源没有描述注解`kubernetes.io/ingress.class`时，`TKE Ingress Controller`会管理当前 Ingress 资源。
- 当 Ingress 资源有注解 `kubernetes.io/ingress.class`且值为`qcloud`时，`TKE Ingress Controller` 会管理当前 Ingress 资源。

详细配置参考：[TKE Ingress Controller 使用方法](https://cloud.tencent.com/document/product/457/45685#ingress-controller-.E4.BD.BF.E7.94.A8.E6.96.B9.E6.B3.95)。

## Nginx Ingress

Nginx Ingress 控制器启动时通过指定`--ingress-class=<INGRESS_CONTROLLER_NAME>`

参数来声明自己监听的 Ingress 配置类范围，其缺省为`nginx` ，Nginx Ingress 控制器指定类名示例如下：

```yaml
...
spec:
  template:
     spec:
       containers:
         - name: nginx-ingress-internal-controller
           args:
             - /nginx-ingress-controller
             - '--ingress-class=<INGRESS_CONTROLLER_NAME>'
...
```

当有 Ingress 资源配置中具有注解 `kubernetes.io/ingress.class: "<INGRESS_CONTROLLER_NAME>"`时将被该控制器监听使用，其 Ingress 资源配置示例如下：

```ymal
...
metadata:
  name: foo
  annotations:
    kubernetes.io/ingress.class: "<INGRESS_CONTROLLER_NAME>"
...
```

# 多个 Ingress 控制器共同使用

根据上述使用配置说明，**建议所有 Ingress 资源都配置注解来区分不同的 Ingress 控制器作用范围，当要使用基于 CLB  的 Ingress 时，配置注解 `kubernetes.io/ingress.class:"qcloud"`，当要使用 Nginx ingress 控制器时配置注解 `kubernetes.io/ingress.class:"<INGRESS_CONTROLLER_NAME>"`（不要和基于CLB Ingress 冲突） 即可。**
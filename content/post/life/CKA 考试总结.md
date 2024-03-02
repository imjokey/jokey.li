+++
author = "Jokey Li"
title = "CKA 考试心得"
date = "2021-12-11"
description = "CKA 考试心得分享"
featured = true
tags = [
    "CKA",
    "CNCF"
]
categories = [
    "生活杂谈",
]
series = ["生活杂谈"]
thumbnail = "images/materials/bathdog.jpeg"
+++


前几天终于把拖了半年多的 CKA 考试预约了，平常由于忙于工作（借口），也没有专门针对复习过，但是实在拖的太久了，哈哈，想着无论如何都要去考，结果就很轻松的考过了，简单记录下考试需要注意的地方和方法。

## 考前准备

### 考试预约

根据官网文档指导进行预约考试即可，参考：[考试预约文档](https://docs.linuxfoundation.org/tc-docs/certification/quick-guide/schedule-an-exam)。

> **重要**：考试时需要有可以证明考生身份的含有拉丁字母的身份证件，一般是护照，鉴于当前的中国的护照办理收紧的现状，如果没有有效期内的护照，刚好可以以考 CKA 考试为由办理护照，本人办理护照的详细经历参考： [疫情政策下顺利办到护照经历](/post/life/2021年疫情政策下顺利办到护照经历/)。

### 考试环境准备

根据官网 tips 指引进行环境电脑考试环境检查，参考 [考试预约文档](https://docs.linuxfoundation.org/tc-docs/certification/quick-guide/schedule-an-exam) 和 [参加考试](https://docs.linuxfoundation.org/tc-docs/certification/quick-guide/taking-an-exam) 文档。

> **重要**：**正式考试时需要 vivaldi 浏览器**，尽管在考试官网环境检查时显示需要 chrome 最新版。 

### 考前模拟

成功报名考试后，**会有两次免费的模拟考试机会**，个人觉得这个模拟考还是很有帮助的，可以提前熟悉下考试的题目类型、题目难度等，但由于之前没经验，考前前一晚上才发现还有模拟题，就仓促做了十几道，但实际上对第二天的考试帮助还是很大的。

### 考试心得

- CKA 考试时只能允许打开 **kubernets.io** 为后缀的域名网站或者是 Github 托管的kubernetes 文档 一般都是直接使用 kubernerts 官网文档，因为可以在任意页面的**左上角搜索栏通过关键字搜索相关内容，可以提升查找资料的效率**。

- 需要注意的是，**在官网文档搜索栏搜索出来的内容链接注意看下是否满足上述允许打开的链接要求，另外大部分题目都是在 [tasks](https://kubernetes.io/docs/tasks/)  中可以找到原题**。

- 正式考试比模拟考试题数少， 模拟考一共25道，由易到难；但是正式考时只有17道题，题目难度还好，后面的题目反而比前面的题目简单好做。

- **读题要仔细**，在做一道 etcd 快照备份、还原题时，因为没有读完题，还费劲的从 master 节点去copy etcd 的证书和 key，实际上人家在worker节点（需要操作的节点）上已经存放了相关认证证书了 ^_^。

- 考试时间相对比较充裕，做题时不要着急，但是遇到不熟悉的题，或者找了半天文档进展不大的题时，建议先直接跳过，不一定所有的题都要做，**最终目的是考过，不是满分**，所以避免浪费太多时间。

- **考试过程中尽量多去找相关 Yaml 然后修改**，这样做题效率是最高的，可以复制粘贴到考试界面提供的文本编辑器中，我当时没找到文本编辑器就直接在命令行 vi了（效率比较低）， 应该是在考试界面右上侧的菜单选项。

- 现在还能想到的考试题目：RBAC 授权，Deployment 挂载 PVC（cbs）， etcd 快照备份和还原，sidecar 标准输出业务容器日志（emptyDir），Nginx Ingress，节点污点和容忍度、静态 Pod、工作负载扩缩容（scale 命令），集群控制面升级，节点不健康修复等。

### 总结

在考之前还有点小担心自己没有系统复习过，但考过了才发现实际上不难，都是一些平常基本的操作知识，大部分在Kubernetes 官网都可以查的到，尤其是 tasks 部分。只要多熟悉下官网文档和仔细读懂题意，对于会点 k8s 基础的同学都没啥问题。下面附上我的CKA 证书：

<object data="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/add1f47b-2677-47da-be8a-49cb41f2e7a0-quanjiang-li-6a8365e8-ac1a-4c98-9dd9-042091dee897-certificate.pdf" type="application/pdf" width="100%">
    <embed src="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/add1f47b-2677-47da-be8a-49cb41f2e7a0-quanjiang-li-6a8365e8-ac1a-4c98-9dd9-042091dee897-certificate.pdf">
        <p>This browser does not support PDFs. Please download the CKA PDF to view it: <a href="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/add1f47b-2677-47da-be8a-49cb41f2e7a0-quanjiang-li-6a8365e8-ac1a-4c98-9dd9-042091dee897-certificate.pdf">Download PDF</a>.</p>
    </embed>
</object>

### 参考链接：

Kubernetes Tasks: https://kubernetes.io/docs/tasks/

CNCF 认证指导文档: https://docs.linuxfoundation.org/tc-docs/certification/quick-guide/
+++
author = "Jokey Li"
title = "使用 rsync 远程增量备份数据"
date = "2020-10-22"
description = "使用 rsync 远程增量备份数据。"
featured = true
tags = [
    "rsync",
]
categories = [
    "实用技巧",
    "数据备份"
]
series = ["网路技术"]
thumbnail = "images/materials/elephent.jpeg"
+++

## 配置备份主机和目标主机的 ssh 免密认证

先生成主机ssh公钥和私钥，并添加主机公钥到远程备份主机的可信任公钥列表：

```bash
ssh-keygen 
cat ~/.ssh/id_rsa.pub | ssh root@remotehost 'cat >> ~/.ssh/authorized_keys'
```

## 执行远程备份

在 shell 环境中执行备份命令或添加备份命令至 `Crond` 配置文件实现周期执行备份，如下示例命令表示：仅递归同步当前主机下的`/sourcedir` 目录下的 `dir1`，`dir2` 目录文件：

```bash
rsync --progress --delete -arHz --include 'dir1/' --include 'dir2/' --exclude '/*' /sourcedir/ root@remotehost:/backup/
```

可根据实际情况变更备份参数，上述的备份命令参数说明：

```bash
-a: 归档文件模式
-r: 递归同步
-H: 建立文件硬链接
-z: 使用压缩文件传输
--progress: 输出同步日志
--delete: 同步删除与原数据不匹配的数据（非常有用）
--include: 包含某个文件或目录
--exclude: 除过某个文件或目录

sourcedir: 原数据目录
remotehost: 目的主机地址
backup: 目的数据目录
```
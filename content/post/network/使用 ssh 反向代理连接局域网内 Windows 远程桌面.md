+++
author = "Jokey Li"
title = "使用 ssh 反向代理连接局域网内 Windows 远程桌面"
date = "2020-09-22"
description = "使用 ssh 反向代理连接局域网内 Windows 远程桌面。"
featured = true
tags = [
    "ssh",
    "Win远程桌面"
]
categories = [
    "实用技巧",
]
series = ["网路技术"]
thumbnail = "images/materials/bathdog.jpeg"
+++


> ⚠️ 此方法会把内网远程桌面服务暴露在外网，安全风险请自行评估！

## 操作步骤

前提条件，当然是需要打开被远程的 Win10 系统的远程桌面的访问功能...，然后在目标 win10  主机上，使用自带的openssh 软件运行`ssh`反向代理命令：

```bash
ssh -R *:3389:127.0.0.1:3389 root@jokeysoft.com
```

就可以通过 “jokeysoft.com:3389” 端口进行远程桌面连接了。

## 进阶用法

上述命令只能临时使用，如果需要系统后台使用并开机自启脚本运行，创建批处理文件`start_ssh_tunnel.bat`键入如下脚本：

```bat
@ECHO OFF
ssh -R *:3389:127.0.0.1:3389 root@jokeysoft.com
PAUSE
```

创建 `start_ssh_tunnel.vbs` 脚本使用静默模式启动（后台运行），如下代码所示：

```vbs
Set WshShell = CreateObject("WScript.Shell") 
WshShell.Run chr(34) & "C:\Users\dell\start_ssh_tunnel.bat" & Chr(34), 0
Set WshShell = Nothing
```

将 `start_ssh_tunnel.vbs` 脚本文件放至 Win10 开机启动脚本目录：

```vbs
当前用户生效目录：
%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup 
全局生效目录:
%ProgramData%\Microsoft\Windows\Start Menu\Programs\StartUp
```

Win10 系统启动脚本目录位置可通过`WindowsKey+R` 输入 `shell:startup` 回车后查看，也可以在`cmd`命令行下使用`explorer shell:startup`命令查看。

**如果长时间与ssh 反向代理主机没有数据包通信，ssh 通信隧道会断开，可以通过配置心跳机制保持 ssh 代理通道不中断：**
ssh 客户端：添加`~/.ssh/config`文件，粘贴如下内容：

```bash
Host *
ServerAliveInterval 60
ServerAliveCountMax 10
```

或者 sshd 服务端修改配置文件`/etc/ssh/sshd_config`：

```bash
ClientAliveInterval 60
ClientAliveCountMax 10
```

至此，启动远程桌面软件输入 ssh 反向代理的外网地址和端口就可以使用 `ssh` 实现的内网穿透来远程局域网的主机了。

![连接成功示例](/images/articles/7kj26b4aik.png)

## 总结

这里抛砖引玉实现了在外网访问局域网内 Win 10 远程桌面服务的反向代理， 实际上 ssh 代理通道可以实现很多类似场景的通信需求，大家按照实际需求开动脑筋使用即可。
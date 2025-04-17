---
title: Windows 11 在 GitLab Runner 服务内找不到 SMB 挂载的 NAS 文件夹
description: 我在 Windows11 上，用服务的方式注册了 GitLab Runner 工具，让  GitLab Runner 工具调度代码执行。在运行过程中，发现我在主机上采用 SMB 挂载的 NAS 文件夹找不到
tags: git
category: 
---

<!-- CreateTime:2025/04/16 07:23:58 -->

<!-- 发布 -->
<!-- 博客 -->

我尝试在 GitLab Runner 服务调度启动的进程里面，使用 `Directory.Exists(@"Y:\")` 这句代码时，发现返回的是文件夹不存在。但我明明就存在 Y 盘这个 SMB 挂载的网络映射驱动器。我尝试自己在机器上直接 dotnet run 运行这句代码时，却很符合预期地返回了文件夹存在。感觉就是 Windows 的服务正在投毒

我在昨天跑得时候还是好好的，但是昨天晚上有一次系统更新，系统更新之后重启机器，我就发现在 GitLab Runner 服务里面调度起来的进程都找不到 SMB 挂载的 NAS 文件夹

在 [lsj](https://blog.sdlsj.net) 伙伴的帮助下，尝试推进了 `net use` 命令到 GitLab Runner 服务调度的新进程里面跑，看到输出信息如下

```
net use
会记录新的网络连接。
状态       本地        远程                      网络
-------------------------------------------------------------------------------
不可用       Y:        \\nas.lindexi.com\Data   Microsoft Windows Network
命令成功完成。
```

再推进 PowerShell 的 powershell Get-SmbMapping 命令，也是输出状态不可用，输出信息如下

```
powershell Get-SmbMapping
Status      Local Path Remote Path            
------      ---------- -----------            
Unavailable Y:         \\nas.lindexi.com\Data
ExitCode: 0
```

相同的命令，我在服务之外，自己用 win+r 打开 PowerShell 运行却是输出好好的，挂载 SMB 文件夹在资源管理器里面也能看到

```
PS C:\Users\lindexi> Get-SmbMapping

Status Local Path Remote Path
------ ---------- -----------
OK     Y:         \\nas.lindexi.com\Data
```

根据 [映射的网络驱动器可能无法在 Windows 10 版本 1809 中重新连接 - Windows Client - Microsoft Learn](https://learn.microsoft.com/zh-cn/troubleshoot/windows-client/networking/mapped-network-drive-fail-reconnect ) 文档可知，这可能是一个已知问题，但这篇文档没有提及服务运行进程的差异

我再次推进挂载命令 `powershell New-SmbMapping -LocalPath Y: -RemotePath \\nas.lindexi.com\Data -Persistent $True` 给到 GitLab Runner 调度的进程，此时输出挂载成功

```
powershell New-SmbMapping -LocalPath Y: -RemotePath \\nas.lindexi.com\Data -Persistent $True
Status Local Path Remote Path            
------ ---------- -----------            
OK     Y:         \\nas.lindexi.com\Data
ExitCode: 0
powershell Get-SmbMapping
Status Local Path Remote Path            
------ ---------- -----------            
OK     Y:         \\nas.lindexi.com\Data
ExitCode: 0
```

这句 New-SmbMapping 命令传入的时候不需要带密码了，也不能重复调用，重复调用会遇到以下错误信息

```
New-SmbMapping : 本地设备名已在使用中。 
所在位置 行:1 字符: 1
+ New-SmbMapping -LocalPath Y: -RemotePath \\nas.lindexi.com\Data -Per ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (MSFT_SmbMapping:ROOT/Microsoft/...MSFT_SmbMapping) [New-SmbMapping], CimE 
   xception
    + FullyQualifiedErrorId : Windows System Error 85,New-SmbMapping
```

执行完成 New-SmbMapping 命令之后，重新在 GitLab Runner 服务调度启动的进程里面，使用 `Directory.Exists(@"Y:\")` 这句代码时，可以正常返回文件夹存在，问题解决

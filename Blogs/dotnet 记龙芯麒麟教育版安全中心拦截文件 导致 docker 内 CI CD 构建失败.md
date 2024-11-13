---
title: dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败
description: 本文记录我在龙芯旧世界的麒麟教育版系统上，使用 docker 对 dotnet 进行构建和打包过程中，因为安全中心拦截而出现奇怪的问题
tags: dotnet
category: 
---

<!-- 发布 -->
<!-- 博客 -->

以下是我打包构建过程中遇到的文件没权限或被占用或 Mutex 失败等信息

使用 GitLab Runner 拉取项目时，遇到的报错信息如下

```
Running with gitlab-runner 13.11.0~beta.6.gc6e248a6 (c6e248a6)
Preparing the "shell" executor
Running on 98c4e471e0d5...
Getting source from Git repository
bash: line 27: /root/build/builds/glrt-9d1/0/t/app/dev/win/LindexiApps.tmp/CI_SERVER_TLS_CA_FILE: Permission denied
Cleaning up file based variables
rm: cannot remove '/root/build/builds/glrt-9d1/0/t/app/dev/win/LindexiApps.tmp/CI_SERVER_TLS_CA_FILE': Permission denied
ERROR: Job failed: exit status 1
```

或者是如下报错信息

```
warning: unable to unlink '/root/build/builds/glrt-9d1/0/t/app/dev/win/LindexiApps.tmp/git-template/config.lock': Permission denied
error: could not write config file /root/build/builds/glrt-9d1/0/t/app/dev/win/LindexiApps.tmp/git-template/config: Permission denied
Cleaning up file based variables
rm: cannot remove '/root/build/builds/glrt-9d1/0/t/app/dev/win/LindexiApps.tmp/CI_SERVER_TLS_CA_FILE': Permission denied
```

使用 dotnet 构建 Avalonia 过程遇到的问题：

```
Avalonia error AVLN9999: Access to the path '/root/build/builds/glrt-9d1/0/t/app/dev/win/LindexiApps/artifacts/obj/LindexiApps/release/LindexiApps.dll' is denied.
```

或者是构建过程中碰到 Mutex 锁需要写入 `/tmp/.dotnet/shm` 都会失败

或者是一些更加看不懂的失败

```
MSBuild version 17.8.5+b5265ef37 for .NET
  Determining projects to restore...
dotnet: pthread_mutex_lock.c:81: __pthread_mutex_lock: Assertion `mutex->__data.__owner == 0' failed.
bash: line 121:   251 Aborted   
```

感谢 [lsj](https://blog.sdlsj.net/ ) 帮忙找到是麒麟教育版安全中心在进行拦截。如下图所示，可以看到很多文件路径都被拦截了

<!-- ![](image/dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败/dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202411/1080237-20241114073113049-1466110711.png)

其中写入到 `/tmp/.dotnet/shm` 路径的，大部分都是和 Mutex 有关

这个拦截不仅拦截 dotnet 系的应用，也拦截 GitLab Runner 应用

按照 [lsj](https://blog.sdlsj.net/ ) 给的教程，这里遇到的是 `kid protect  : disable_privacy` 拦截。通过上图的 KYSEC_KID 类型即可了解到。这里的 KYSEC 是麒麟系统安全机制，详细请参阅 [kylinos-kysec介绍 - gpysir - 博客园](https://www.cnblogs.com/gpysir/p/15165757.html )

在物理机终端输入 `$ sudo getstatus` 即可看到如下输出内容

```
$ sudo getstatus
KySec status: enabled
 
exec control : warning
net control  : off
file protect : off
kmod protect : off
three admin  : off
process protect: off
device control: off
ipt control  : off
kid protect  : disable_privacy
program blklist: off
eperm control: off
```

如上面输出内容 `KySec status: enabled` 可以看到是开启状态。且通过 `kid protect  : disable_privacy` 可以看到开启了 kid protect 功能，也就是 KYSEC_KID 类型

尝试单独关闭 kid 功能，如下面命令

```
$ sudo setstatus -f kid off
```

禁用之后输入 `$ sudo getstatus` 即可看到如下输出内容

```
$ sudo setstatus -f kid off
$ sudo getstatus
KySec status: enabled
 
exec control : warning
net control  : off
file protect : off
kmod protect : off
three admin  : off
process protect: off
device control: off
ipt control  : off
kid protect  : off
program blklist: off
eperm control: off
```

如此可以看到没有完全关闭麒麟系统安全机制，只是关闭了 kid protect 功能

详细请参阅 [银河麒麟系统安全机制-KYSEC - 多弗朗强哥 - 博客园](https://www.cnblogs.com/chendeqiang/p/15173757.html )

重新尝试构建，此时所有的构建都符合预期。无论是 dotnet 还是 GitLab Runner 都能跑起来

这个问题坑了我一天多的时间，太感谢 [lsj](https://blog.sdlsj.net/ ) 的帮助了

吐槽点是这里的拦截没有什么界面可以看到关闭选项，拦截的过程中也没有给出通知，遇到这些问题我猜不到是被拦截

以下是我拿到的龙芯麒麟教育版系统的信息

```bash
$ uname -a
Linux lindexi-pc 5.4.18-116-generic #105-KYLINOS SMP Fri Jun 21 14:09:22 UTC 2024 loongarch64 loongarch64 loongarch64 GNU/Linux
```

```bash
$ cat /etc/os-release
NAME="Kylin"
VERSION="银河麒麟桌面操作系统（教育版）V10"
VERSION_US="Kylin Linux Desktop EDU V10"
ID=kylin
ID_LIKE=debian
PRETTY_NAME="Kylin V10 SP1"
VERSION_ID="v10"
HOME_URL="http://www.kylinos.cn/"
SUPPORT_URL="http://www.kylinos.cn/support/technology.html"
BUG_REPORT_URL="http://www.kylinos.cn/"
PRIVACY_POLICY_URL="http://www.kylinos.cn"
VERSION_CODENAME=kylin
UBUNTU_CODENAME=kylin
PROJECT_CODENAME=V10SP1-General-Edu
KYLIN_RELEASE_ID="2403"
```

```bash
$ cat /etc/debian_version
bullseye/sid
```

```bash
$ cat /etc/kylin-version/kylin-system-version.conf
[SYSTEM]
os_version = 2403
update_version = 2403
quality_version =
```

```bash
$ cat /etc/.kyinfo
[dist]
name=Kylin-Desktop-EDU
milestone=V10
arch=loongarch64
beta=False
time=2024-09-14 12:27:59
dist_id=Kylin-Desktop-V10-SP1-2403-update1-EDU-Release-20240914-LoongArch64-2024-09-14 12:27:59

[servicekey]
key=0389218

[os]
to=
term=2025-12-18
```

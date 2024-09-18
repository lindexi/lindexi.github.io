---
title: dotnet 8 版本与银河麒麟V10和UOS系统的 glibc 兼容性
description: 刚刚好 dotnet 8 的 glibc 版本足够旧，可以运行

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/04/26 18:38:48 -->

<!-- 发布 -->
<!-- 博客 -->

本文记录于 2024.04.26 如果你阅读本文时间距离本文记录时间过远，可能本文记录的信息已失效

## dotnet

根据 dotnet 的 [supported-os](https://github.com/dotnet/core/blob/79c78c7261069cb2953835044336e63a8d524ca1/release-notes/8.0/supported-os.md) 文档记录，当前的 dotnet 8 是 8.0.4 版本，官方说明是支持 Debian 11 及以上版本

实际测试可以在 debian 10 运行，且构建打包制作 AOT 等也正常

构建 dotnet 8 使用的是 Ubuntu 16.04 系统，依赖的 glibc 是 2.23 版本

## debian 10

debian 10.13 版本的 glibc 版本是 2.28 版本

通过 `ldd --version` 命令行获取的 glibc 版本信息如下

```
ldd (Debian GLIBC 2.28-10+deb10u2) 2.28
```

可见 debian 10.13 的 glibc 版本大于 dotnet 8 的依赖 glibc 版本，从基础层面上提供了可运行的可能

经过我的稍微复杂的测试，发现了 dotnet 8 可以运行起来，且 AOT 部分也正常

## 麒麟

在我的设备上使用 `cat /etc/.kyinfo` 获取麒麟系统的版本的输出信息如下

```
[dist]
name=Kylin
milestone=Desktop-V10-SP1-General-Release-TSM-lindexi-20230217
arch=arm64
beta=False
time=2023-02-17 19:01:29
```

根据 [定昌电子](https://www.gzdcsmt.com/sys-nd/1193.html ) 记录的文档，这里的 Desktop V10 SP1 General Release 版本就是银河麒麟桌面操作系统V10 SP1版本

运行 `uname -r` 的输出如下

```
>$ uname -r
5.4.18-53sy01-generic
```

在麒麟系统上运行 `cat /etc/debian_version` 获取 debian 版本号，输出信息如下

```
>$ cat /etc/debian_version
bullseye/sid
```

bullseye 是 debian 11 的发布代号，详细请看 <https://www.debian.org/releases/bullseye/>

运行 `ldd --version` 命令行获取的 glibc 版本信息如下

```
>$ ldd --version
ldd (Ubuntu GLIBC 2.31-0kylin9.1k20.5) 2.31
```

可以看到麒麟V10基于 debian 11 且 glibc 为 2.31 版本，完全在 dotnet 8 支持范围内

## UOS

在我的 UOS 虚拟机上运行 `cat /etc/product-info` 获取 UOS 的版本号信息的输出如下

```
>$ cat /etc/product-info
UnionTech OS-20-20221214083720_x86_64_E_1050_4-19
```

运行 `cat /etc/os-version` 查看 UOS 统信操作系统版本的输出如下

```
>$ cat /etc/os-version
[Version]
SystemName=UnionTech OS Desktop
SystemName[zh_CN]=统信桌面操作系统
ProductType=Desktop
ProductType[zh_CN]=桌面
EditionName=E
EditionName[zh_CN]=E
MajorVersion=20
MinorVersion=1050
OsBuild=11068.102
```

运行 `cat /etc/os-release` 的输出如下

```
>$ cat /etc/os-release
PRETTY_NAME="UnionTech OS Desktop 20 E"
NAME="uos"
VERSION_ID="20"
VERSION="20"
ID=uos
HOME_URL="https://www.chinauos.com/"
BUG_REPORT_URL="http://bbs.chinauos.com"
VERSION_CODENAME=uranus
```

运行 `uname -r` 的输出如下

```
>$ uname -r
4.19.0-amd64-desktop
```

运行 `cat /etc/debian_version` 获取 debian 版本号，输出信息如下

```
>$ cat /etc/debian_version
10.10
```

运行 `ldd --version` 命令行获取的 glibc 版本信息如下

```
>$ ldd --version
ldd (Debian GLIBC 2.28.19-1+dde) 2.28
```

可以看到 UOS 的 OS-20-20221214083720 20.1050.11068.102 版本是基于 debian 10 的，在 dotnet 8 官方声明的支持范围之外。但 glibc 的版本是 2.28 版本，大于 dotnet 8 的依赖版本，提供了基础运行的可能。经过我的稍微复杂的测试，发现了 dotnet 8 可以运行起来，且 AOT 部分也正常

更多 Linux 和国产系统的开发相关博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

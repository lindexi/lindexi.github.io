---
title: 修复 Debian 安装 dotnet 失败 depends on ca-certificates
description: 本文记录我在 Debian 安装 dotnet 失败，报错信息是 packages-microsoft-prod depends on ca-certificates; however:  Package ca-certificates is not installed.
tags: dotnet
category: 
---

<!-- CreateTime:2024/2/28 14:30:37 -->

<!-- 发布 -->
<!-- 博客 -->

一开始按照官方的以下代码例子进行安装 packages-microsoft-prod.deb 文件，命令如下

```
sudo dpkg -i packages-microsoft-prod.deb
```

报错信息如下

```
(Reading database ... 9939 files and directories currently installed.)
Preparing to unpack packages-microsoft-prod.deb ...
Unpacking packages-microsoft-prod (1.1-debian12) over (1.1-debian12) ...
dpkg: dependency problems prevent configuration of packages-microsoft-prod:
 packages-microsoft-prod depends on ca-certificates; however:
  Package ca-certificates is not installed.
```

根据网上提供的命令，通过以下命令重新安装 ca-certificates 工具

```
sudo apt-get install --reinstall ca-certificates
```

依然提示失败，这次是依赖的 openssl 没有安装，错误提示如下

```
Reading package lists... Done
Building dependency tree... Done
You might want to run 'apt --fix-broken install' to correct these.
The following packages have unmet dependencies:
 ca-certificates : Depends: openssl (>= 1.1.1) but it is not going to be installed
E: Unmet dependencies. Try 'apt --fix-broken install' with no packages (or specify a solution)
```

解决方法：

先通过 `apt --fix-broken install` 命令解开错误，输入命令行如下

```
sudo apt --fix-broken install
```

运行成功，输出如下

```
Reading package lists... Done
Building dependency tree... Done
Correcting dependencies... Done
The following additional packages will be installed:
  ca-certificates libssl1.1 openssl
The following NEW packages will be installed:
  ca-certificates libssl1.1 openssl
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
1 not fully installed or removed.
Need to get 2,573 kB of archives.
After this operation, 6,112 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mirrors.aliyun.com/debian-security buster/updates/main amd64 libssl1.1 amd64 1.1.1n-0+deb10u6 [1,552 kB]
Get:2 http://mirrors.aliyun.com/debian-security buster/updates/main amd64 openssl amd64 1.1.1n-0+deb10u6 [855 kB]
Get:3 http://mirrors.aliyun.com/debian buster/main amd64 ca-certificates all 20200601~deb10u2 [166 kB]
Fetched 2,573 kB in 3s (930 kB/s)
Preconfiguring packages ...
Selecting previously unselected package libssl1.1:amd64.
(Reading database ... 9939 files and directories currently installed.)
Preparing to unpack .../libssl1.1_1.1.1n-0+deb10u6_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.1n-0+deb10u6) ...
Selecting previously unselected package openssl.
Preparing to unpack .../openssl_1.1.1n-0+deb10u6_amd64.deb ...
Unpacking openssl (1.1.1n-0+deb10u6) ...
Selecting previously unselected package ca-certificates.
Preparing to unpack .../ca-certificates_20200601~deb10u2_all.deb ...
Unpacking ca-certificates (20200601~deb10u2) ...
Setting up libssl1.1:amd64 (1.1.1n-0+deb10u6) ...
Setting up openssl (1.1.1n-0+deb10u6) ...
Setting up ca-certificates (20200601~deb10u2) ...
Updating certificates in /etc/ssl/certs...
137 added, 0 removed; done.
Setting up packages-microsoft-prod (1.1-debian12) ...
Processing triggers for libc-bin (2.36-9) ...
Processing triggers for ca-certificates (20200601~deb10u2) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

接着使用以下命令重新安装 ca-certificates 工具

```
sudo apt-get --fix-broken install --reinstall ca-certificates
```

安装成功，输出如下

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 1 reinstalled, 0 to remove and 0 not upgraded.
Need to get 166 kB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/debian buster/main amd64 ca-certificates all 20200601~deb10u2 [166 kB]
Fetched 166 kB in 0s (789 kB/s)
Preconfiguring packages ...
(Reading database ... 10266 files and directories currently installed.)
Preparing to unpack .../ca-certificates_20200601~deb10u2_all.deb ...
Unpacking ca-certificates (20200601~deb10u2) over (20200601~deb10u2) ...
Setting up ca-certificates (20200601~deb10u2) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Processing triggers for ca-certificates (20200601~deb10u2) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

再继续安装 `packages-microsoft-prod.deb` 使用以下命令

```
sudo dpkg -i packages-microsoft-prod.deb
```

执行成功，输出如下

```
(Reading database ... 10266 files and directories currently installed.)
Preparing to unpack packages-microsoft-prod.deb ...
Unpacking packages-microsoft-prod (1.1-debian12) over (1.1-debian12) ...
Setting up packages-microsoft-prod (1.1-debian12) ...
File /usr/share/keyrings/microsoft-prod.gpg is missing. Installing...
```

接着执行 `sudo apt-get update` 命令

```
sudo apt-get update
```

执行成功，输出如下

```
Get:1 https://packages.microsoft.com/debian/12/prod bookworm InRelease [3,617 B]
Hit:2 http://mirrors.aliyun.com/debian buster InRelease
Get:3 https://packages.microsoft.com/debian/12/prod bookworm/main armhf Packages [5,346 B]
Get:4 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 Packages [29.1 kB]
Get:5 https://packages.microsoft.com/debian/12/prod bookworm/main all Packages [342 B]
Get:6 https://packages.microsoft.com/debian/12/prod bookworm/main arm64 Packages [5,932 B]
Get:7 http://mirrors.aliyun.com/debian-security buster/updates InRelease [34.8 kB]
Hit:8 http://mirrors.aliyun.com/debian buster-updates InRelease
Get:9 http://mirrors.aliyun.com/debian buster-backports InRelease [51.4 kB]
Get:10 http://mirrors.aliyun.com/debian-security buster/updates/main Sources [371 kB]
Get:11 http://mirrors.aliyun.com/debian-security buster/updates/main amd64 Packages [586 kB]
Get:12 http://mirrors.aliyun.com/debian-security buster/updates/main Translation-en [318 kB]
Get:13 http://mirrors.aliyun.com/debian buster-backports/main Sources.diff/Index [27.8 kB]
Get:14 http://mirrors.aliyun.com/debian buster-backports/main amd64 Packages.diff/Index [27.8 kB]
Get:15 http://mirrors.aliyun.com/debian buster-backports/main Sources 2024-02-11-0807.14.pdiff [1,391 B]
Get:16 http://mirrors.aliyun.com/debian buster-backports/main Sources 2024-02-11-1408.12.pdiff [33 B]
Get:16 http://mirrors.aliyun.com/debian buster-backports/main Sources 2024-02-11-1408.12.pdiff [33 B]
Get:17 http://mirrors.aliyun.com/debian buster-backports/main amd64 Packages 2024-02-11-1408.12.pdiff [1,249 B]
Get:17 http://mirrors.aliyun.com/debian buster-backports/main amd64 Packages 2024-02-11-1408.12.pdiff [1,249 B]
Fetched 1,464 kB in 2s (684 kB/s)
Reading package lists... Done
```

接着执行安装 .NET 7 SDK 命令

```
sudo apt-get install -y dotnet-sdk-7.0
```

执行成功，输出如下

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  aspnetcore-runtime-7.0 aspnetcore-targeting-pack-7.0 dotnet-apphost-pack-7.0 dotnet-host dotnet-hostfxr-7.0
  dotnet-runtime-7.0 dotnet-runtime-deps-7.0 dotnet-targeting-pack-7.0 libicu63 netstandard-targeting-pack-2.1
The following NEW packages will be installed:
  aspnetcore-runtime-7.0 aspnetcore-targeting-pack-7.0 dotnet-apphost-pack-7.0 dotnet-host dotnet-hostfxr-7.0
  dotnet-runtime-7.0 dotnet-runtime-deps-7.0 dotnet-sdk-7.0 dotnet-targeting-pack-7.0 libicu63
  netstandard-targeting-pack-2.1
0 upgraded, 11 newly installed, 0 to remove and 0 not upgraded.
Need to get 156 MB of archives.
After this operation, 616 MB of additional disk space will be used.
Get:1 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-runtime-deps-7.0 amd64 7.0.16-1 [2,890 B]
Get:2 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-host amd64 8.0.2-1 [37.6 kB]
Get:3 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-hostfxr-7.0 amd64 7.0.16-1 [145 kB]
Get:4 http://mirrors.aliyun.com/debian buster/main amd64 libicu63 amd64 63.1-6+deb10u3 [8,293 kB]
Get:5 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-runtime-7.0 amd64 7.0.16-1 [23.2 MB]
Get:6 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 aspnetcore-runtime-7.0 amd64 7.0.16-1 [7,065 kB]
Get:7 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-targeting-pack-7.0 amd64 7.0.16-1 [2,568 kB]
Get:8 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 aspnetcore-targeting-pack-7.0 amd64 7.0.16-1 [1,525 kB]
Get:9 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-apphost-pack-7.0 amd64 7.0.16-1 [3,523 kB]
Get:10 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 netstandard-targeting-pack-2.1 amd64 2.1.0-1 [1,476 kB]
Get:11 https://packages.microsoft.com/debian/12/prod bookworm/main amd64 dotnet-sdk-7.0 amd64 7.0.406-1 [108 MB]
Fetched 156 MB in 2min 1s (1,291 kB/s)
Selecting previously unselected package libicu63:amd64.
(Reading database ... 10266 files and directories currently installed.)
Preparing to unpack .../00-libicu63_63.1-6+deb10u3_amd64.deb ...
Unpacking libicu63:amd64 (63.1-6+deb10u3) ...
Selecting previously unselected package dotnet-runtime-deps-7.0.
Preparing to unpack .../01-dotnet-runtime-deps-7.0_7.0.16-1_amd64.deb ...
Unpacking dotnet-runtime-deps-7.0 (7.0.16-1) ...
Selecting previously unselected package dotnet-host.
Preparing to unpack .../02-dotnet-host_8.0.2-1_amd64.deb ...
Unpacking dotnet-host (8.0.2-1) ...
Selecting previously unselected package dotnet-hostfxr-7.0.
Preparing to unpack .../03-dotnet-hostfxr-7.0_7.0.16-1_amd64.deb ...
Unpacking dotnet-hostfxr-7.0 (7.0.16-1) ...
Selecting previously unselected package dotnet-runtime-7.0.
Preparing to unpack .../04-dotnet-runtime-7.0_7.0.16-1_amd64.deb ...
Unpacking dotnet-runtime-7.0 (7.0.16-1) ...
Selecting previously unselected package aspnetcore-runtime-7.0.
Preparing to unpack .../05-aspnetcore-runtime-7.0_7.0.16-1_amd64.deb ...
Unpacking aspnetcore-runtime-7.0 (7.0.16-1) ...
Selecting previously unselected package dotnet-targeting-pack-7.0.
Preparing to unpack .../06-dotnet-targeting-pack-7.0_7.0.16-1_amd64.deb ...
Unpacking dotnet-targeting-pack-7.0 (7.0.16-1) ...
Selecting previously unselected package aspnetcore-targeting-pack-7.0.
Preparing to unpack .../07-aspnetcore-targeting-pack-7.0_7.0.16-1_amd64.deb ...
Unpacking aspnetcore-targeting-pack-7.0 (7.0.16-1) ...
Selecting previously unselected package dotnet-apphost-pack-7.0.
Preparing to unpack .../08-dotnet-apphost-pack-7.0_7.0.16-1_amd64.deb ...
Unpacking dotnet-apphost-pack-7.0 (7.0.16-1) ...
Selecting previously unselected package netstandard-targeting-pack-2.1.
Preparing to unpack .../09-netstandard-targeting-pack-2.1_2.1.0-1_amd64.deb ...
Unpacking netstandard-targeting-pack-2.1 (2.1.0-1) ...
Selecting previously unselected package dotnet-sdk-7.0.
Preparing to unpack .../10-dotnet-sdk-7.0_7.0.406-1_amd64.deb ...
Unpacking dotnet-sdk-7.0 (7.0.406-1) ...
Setting up dotnet-host (8.0.2-1) ...
Setting up netstandard-targeting-pack-2.1 (2.1.0-1) ...
Setting up libicu63:amd64 (63.1-6+deb10u3) ...
Setting up dotnet-targeting-pack-7.0 (7.0.16-1) ...
Setting up dotnet-apphost-pack-7.0 (7.0.16-1) ...
Setting up dotnet-hostfxr-7.0 (7.0.16-1) ...
Setting up dotnet-runtime-deps-7.0 (7.0.16-1) ...
Setting up aspnetcore-targeting-pack-7.0 (7.0.16-1) ...
Setting up dotnet-runtime-7.0 (7.0.16-1) ...
Setting up aspnetcore-runtime-7.0 (7.0.16-1) ...
Setting up dotnet-sdk-7.0 (7.0.406-1) ...
Processing triggers for libc-bin (2.36-9) ...
```

安装完成，输出 dotnet 命令测试一下

```
dotnet
```

输出如下代表成功

```
Usage: dotnet [options]
Usage: dotnet [path-to-application]

Options:
  -h|--help         Display help.
  --info            Display .NET information.
  --list-sdks       Display the installed SDKs.
  --list-runtimes   Display the installed runtimes.

path-to-application:
  The path to an application .dll file to execute.
```

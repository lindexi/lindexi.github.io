
# 制作一个龙芯旧世界的 dotnet sdk docker 镜像

本文将和大家分享如何制作一个在龙芯旧世界上可跑的 dotnet sdk docker 镜像，以及我的踩坑过程

<!--more-->


<!-- CreateTime:2024/11/02 07:29:52 -->

<!-- 发布 -->
<!-- 博客 -->

以下是我的 dockerfile 文件，内容特别简单

```
FROM cr.loongnix.cn/library/debian:buster
WORKDIR /root
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        curl \
        git \
        wget \
        tzdata \
        openssh-client \
        xz-utils \
        libicu67

RUN wget http://ftp.loongnix.cn/dotnet/8.0.7/8.0.7-1/pkg/dotnet-sdk-8.0.107-linux-loongarch64.tar.xz

RUN mkdir dotnet && tar -xvf dotnet-sdk-8.0.107-linux-loongarch64.tar.xz -C dotnet

RUN ln -s /root/dotnet/dotnet /usr/bin/dotnet
```

我这里的 `cr.loongnix.cn/library/debian:buster` 的 docker image Id 是 70fcf9ce129a，属于龙芯旧世界版本

以上 dockerfile 在 2024.11.01 时能正常构建且构建出来的 docker image 可以在龙芯旧世界里使用。以上的 dockerfile 的使用方法是在龙芯旧世界设备上，用 docker build 一下即可。特别说明，要在龙芯旧世界设备上，用 docker build 一下，因为 `cr.loongnix.cn/library/debian:buster` 里面使用的就已经是龙架构的旧世界的系统了，在其他 CPU 架构上的设备是无法直接跑起来的

如果大家不想自己构建，可以发邮件向我要 dockerimage-loongarch64-abi1.0-dotnet-sdk-8.0.107-debian-buster.8346c670d9f6.tar 文件，这个文件就是我用以上 dockerfile 构建出来的

以下是我的踩坑记录：

## 一些概念

- 龙芯新旧世界： [旧世界与新世界 - 咱龙了吗？](https://areweloongyet.com/docs/old-and-new-worlds/ )

业界共识是迟早龙芯都会进入新世界，即可能后续的 `cr.loongnix.cn/library/debian:buster` 会是属于新世界的。尽管现在似乎龙芯准备将 `cr.loongnix.cn` 当成旧世界仓库，将 `lcr.loongnix.cn` 和 `https://hub.docker.com/u/loongarch64` 当成新世界仓库

但必须说明的是，我没有从龙芯任何文档找到说一定会将 `cr.loongnix.cn` 当成旧世界仓库的说法，只是从 [龙芯云 https://loongson-cloud-community.github.io/Loongson-Cloud-Community/](https://loongson-cloud-community.github.io/Loongson-Cloud-Community/ ) 文档找到以下描述，从而进行推测而已：

> cr.loongnix.cn 容器镜像仓库, 适用于内核版本 < 5.10的操作系统，如龙蜥8.8、龙芯debian10等。容器镜像源码仓库：<https://github.com/Loongson-Cloud-Community/dockerfiles> 
>
> lcr.loongnix.cn 容器镜像仓库，适用于内核版本>=5.10的操作系统，如欧拉22.03、欧拉24.03、龙蜥23版本等。容器镜像源码仓库：<https://github.com/Loongson-Cloud-Community/docker-library>

再根据 <https://github.com/Loongson-Cloud-Community/docker-library/blob/b14dda076c0deadebe0a949a8603f07c8cc922cb/README.md> 文档的以下内容证明至少 lcr.loongnix.cn 是新世界仓库

> 适用于上游（abi2.0）操作系统的镜像源码仓库,对应的镜像仓库是lcr.loongnix.cn

如 [旧世界与新世界 - 咱龙了吗？](https://areweloongyet.com/docs/old-and-new-worlds/ ) 文档说明，上游（abi2.0）操作系统属于新世界，而 abi1.0 属于旧世界

想要在旧世界运行，就需要找到正确的旧世界仓库。根据上文描述，在 docker hub 上找到的，预计都属于新世界，而不是旧世界

## 在龙芯旧世界 kylin-2403 sp1 安装 docker

我只借到一台宝贵的龙芯设备，这台是 3A6000 的设备，搭载了 kylin-2403 sp1 麒麟系统。系统版本信息内容如下

```bash
$ uname -a
Linux lindexi-pc 5.4.18-116-generic #105-KYLINOS SMP Fri Jun 21 14:09:22 UTC 2024 loongarch64 loongarch64 loongarch64 GNU/Linux
```

```bash
$ uname -r
5.4.18-116-generic
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

```bash
$ cat /etc/debian_version
bullseye/sid
```

CPU 信息如下

```bash
$ cat /proc/cpuinfo | grep name
model name              : Loongson-3A6000
```

```bash
$ lscpu
Architecture:        loongarch64
```

我的 docker 安装方法：

先下载安装 docker-ce-cli：https://pkg.loongnix.cn/loongnix/pool/main/d/docker-ce/docker-ce-cli_20.10.3-7.1_loongarch64.deb

```bash
$ wget https://pkg.loongnix.cn/loongnix/pool/main/d/docker-ce/docker-ce-cli_20.10.3-7.1_loongarch64.deb
$ sudo dpkg -i docker-ce-cli_20.10.3-7.1_loongarch64.deb
```

此时 docker 尚未启动，运行将会提示 `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?` 错误，需要再按照 <https://bbs.loongarch.org/d/80-docker-buildx-compose> 提供的方法进行安装 docker ce 版本

```bash
# docker binaries
$ wget https://github.com/wojiushixiaobai/docker-ce-binaries-loongarch64/releases/download/v24.0.2/docker-24.0.2.tgz
$ tar -xf docker-24.0.2.tgz
$ cp docker/* /usr/local/bin/

# systemd
$ wget https://raw.githubusercontent.com/jumpserver/installer/9dfdfad3d71759483f12a90d62c609ba6f64bc46/scripts/docker/docker.service -O /etc/systemd/system/docker.service
```

启动 docker：

```bash
$ systemctl start docker
```

此时运行 `sudo docker ps` 如能看到以下输出内容，则证明安装成功

```bash
$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

接下来尝试构建上文提供的 dockerfile 文件

```bash
# 先使用 cd 进入到 dockerfile 文件所在目录
$ sudo docker build -t t1 .
```

构建完成之后，试试进入 docker image 里面。以上命令行将构建出 `t1:latest` 的 docker image

```bash
$ sudo docker run -it t1
```

进入 docker 容器之后，试试输入 `dotnet --info` 和 `dotnet nuget --version` 测试 dotnet 是否部署成功

```bash
root@0957d906898b:~# dotnet --info
.NET SDK:
 Version:           8.0.107
 Commit:            1bdaef7265
 Workload version:  8.0.100-manifests.43c23f91

Runtime Environment:
 OS Name:     Loongnix
 OS Version:  20
 OS Platform: Linux
 RID:         linux-loongarch64
 Base Path:   /root/dotnet/sdk/8.0.107/

.NET workloads installed:
 Workload version: 8.0.100-manifests.43c23f91
There are no installed workloads to display.

Host:
  Version:      8.0.7
  Architecture: loongarch64
  Commit:       dbda949b78

.NET SDKs installed:
  8.0.107 [/root/dotnet/sdk]

.NET runtimes installed:
  Microsoft.AspNetCore.App 8.0.7 [/root/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.NETCore.App 8.0.7 [/root/dotnet/shared/Microsoft.NETCore.App]

Other architectures found:
  None

Environment variables:
  Not set

global.json file:
  Not found

Learn more:
  https://aka.ms/dotnet/info

Download .NET:
  http://www.loongnix.cn/zh/api/dotnet

Loongson's .NET version:
  8.0.107-1
```

```bash
root@0957d906898b:~# dotnet nuget --version
NuGet Command Line
6.8.1.32767
```

如能够完成上述步骤，则证明已经成功完成

以下是记录我过程中的一些踩坑点：

其实核心踩坑点是由于龙芯新旧世界的存在，让我不能确定在网上找到的文章属于新世界还是旧世界，再加上我开始对龙芯新旧世界没有概念或概念混乱，导致我踩坑比较多

### docker 源

针对旧世界，不能使用 lcr.loongnix.cn 和 docker hub 源，这两个源都不符合旧世界的预期

也即如 `loongsongd/debian10_loongarch64_x64:mini` 等 docker image 都是不符合预期的。其中 `loongsongd/debian10_loongarch64_x64:mini` 存在比较大的问题是似乎容器里面已经开了 qemu 模拟了，但各个进程，如 bash 等都是非龙架构的，导致运行必定失败 

### docker ce 的安装

不能使用通用的 docker ce 版本，本身龙芯使用的是 龙架构。敲黑板，龙架构的地位和 x86、x86_64 是等同的，即大家可以说，世界上的 CPU 架构有： x86、x64、arm、arm64、龙架构、MIPS

请必须明确的是龙架构是一个独立的架构，不是 MIPS 架构，细节请参阅： [龙芯自主指令集LoongArch为什么是全新指令集，和MIPS不同在哪儿？- 辟谣 - 哔哩哔哩](https://www.bilibili.com/read/cv27388271/ )

更多请参考： [如何称呼龙架构？ - 咱龙了吗？](https://areweloongyet.com/docs/loong-or-loongarch/ )

这就是为什么需要使用龙架构版的 docker ce 的原因了。再叠加新旧世界的影响，就需要使用龙架构的旧世界的 docker ce 包

官方提供的 `https://pkg.loongnix.cn/loongnix/pool/main/d/docker-ce/docker-ce_20.10.3-7.1_loongarch64.deb` 需要有 containerd.io 依赖，不太好装。这就是为什么我去使用论坛提供的安装方法的原因

但是由于 <https://bbs.loongarch.org/d/80-docker-buildx-compose> 论坛上提供的内容比较旧了，一些链接，如 `https://raw.githubusercontent.com/jumpserver/installer/master/scripts/docker.service` 已经 404 了，需要更新

更新之后的命令如下，和上文给出的相同

```bash
# docker binaries
$ wget https://github.com/wojiushixiaobai/docker-ce-binaries-loongarch64/releases/download/v24.0.2/docker-24.0.2.tgz
$ tar -xf docker-24.0.2.tgz
$ cp docker/* /usr/local/bin/

# systemd
$ wget https://raw.githubusercontent.com/jumpserver/installer/9dfdfad3d71759483f12a90d62c609ba6f64bc46/scripts/docker/docker.service -O /etc/systemd/system/docker.service
```

## 哪个才是龙芯官方 GitHub 组织

- https://github.com/Loongson-Cloud-Community ：官方组织
- https://github.com/Loongson-Community ： 非官方的社区组织
- https://github.com/loongson ：非官方的社区组织

根据 [容器镜像仓库 - 龙芯开源社区](https://www.loongnix.cn/zh/cloud/container-registry/ ) 提及的源码仓库为 ： <https://github.com/Loongson-Cloud-Community> ，可认为 Loongson-Cloud-Community 为官方组织

基本上只有官方组织在维护 abi1.0 旧世界，其他组织都是在做新世界。新世界的 dotnet 是在龙芯社区组织里面的，详细请看 <https://github.com/loongson-community/dotnet-unofficial-build>

## dotnet 找不到 icu 不支持多语言

执行 `dotnet --info` 的报错如下

```bash
root@55154e73f795:~# dotnet --info
Process terminated. Couldn't find a valid ICU package installed on the system. Please install libicu (or icu-libs) using your package manager and try again. Alternatively you can set the configuration flag System.Globalization.Invariant to true if you want to run with no globalization support. Please see https://aka.ms/dotnet-missing-libicu for more information.
   at System.Environment.FailFast(System.String)
   at System.Globalization.GlobalizationMode+Settings..cctor()
   at System.Globalization.CultureData.CreateCultureWithInvariantData()
   at System.Globalization.CultureData.get_Invariant()
   at System.Globalization.CultureInfo..cctor()
   at System.Globalization.CultureInfo.get_CurrentUICulture()
   at System.TimeZoneInfo.GetUtcStandardDisplayName()
   at System.TimeZoneInfo.CreateUtcTimeZone()
   at System.TimeZoneInfo..cctor()
   at System.DateTime.get_Now()
   at Microsoft.DotNet.Cli.Program.Main(System.String[])
Aborted (core dumped)
```

通过 apt search libicu 可知道在龙芯源上带了 libicu67 和 libicu63 版本

```bash
root@55154e73f795:~# apt search libicu
...
libicu63/Debian 63.1-6+deb10u1.lnd.1 loongarch64
  International Components for Unicode

libicu67/Debian 67.1-7.lnd.1 loongarch64
  International Components for Unicode
...
```

只需在 dockerfile 带上 `libicu67` 的安装即可解决在龙芯设备上 dotnet 找不到多语言抛出异常的问题，带上之后的 dockerfile 安装内容如下，以下代码和本文一开始给出的 dockerfile 内容相同

```
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        curl \
        git \
        wget \
        tzdata \
        openssh-client \
        xz-utils \
        libicu67
```

以上的 `ca-certificates` 也是必要项，详细请参阅 [修复 Debian 安装 dotnet 失败 depends on ca-certificates](https://blog.lindexi.com/post/%E4%BF%AE%E5%A4%8D-Debian-%E5%AE%89%E8%A3%85-dotnet-%E5%A4%B1%E8%B4%A5-depends-on-ca-certificates.html )

更多多语言相关，请参阅： [Globalization config settings - .NET - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/globalization )

## dotnet 软链接

1. 应该使用 `ln -s` 软链接的方式，否则将会遇到 `Error: [/usr/bin/host/fxr] does not exist` 错误。详细请看 [彻底明白Linux硬链接和软链接-linux硬链接和软链接区别](https://www.51cto.com/article/702714.html )
2. 软链接过程中，不应该使用相对路径，即 `ln -s ./dotnet/dotnet /usr/bin/dotnet` 也是错误的，将会提示 `bash: /usr/bin/dotnet: Too many levels of symbolic links` 错误

详细请看 [制作一个能构建 dotnet AOT 的 gitlab ruuner 的 Debian docker 镜像](https://blog.lindexi.com/post/%E5%88%B6%E4%BD%9C%E4%B8%80%E4%B8%AA%E8%83%BD%E6%9E%84%E5%BB%BA-dotnet-AOT-%E7%9A%84-gitlab-ruuner-%E7%9A%84-Debian-docker-%E9%95%9C%E5%83%8F.html )
<!-- [制作一个能构建 dotnet AOT 的 gitlab runner 的 Debian docker 镜像 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18164886 ) -->

## 为何不使用 dotnet deb 包

从 <http://ftp.loongnix.cn/dotnet/8.0.7/8.0.7-1/deb/dotnet-sdk-8.0_8.0.107-1_loongarch64.deb> 下载的包是需要带依赖的，通过 dpkg -e 获取 control 文件，可见如下内容

```
Depends: dotnet-runtime-8.0 (>= 8.0.7), dotnet-targeting-pack-8.0 (>= 8.0.7), dotnet-apphost-pack-8.0 (>= 8.0.7), netstandard-targeting-pack-2.1 (>= 2.1.0), aspnetcore-runtime-8.0 (>= 8.0.7), aspnetcore-targeting-pack-8.0 (>= 8.0.7)
```

这里有比较多依赖包，不如直接使用 tar.xz 包

## x86_64 配合 qemu 模拟运行龙架构 docker

结论： 不可行

能够跑起来龙架构旧世界的 docker 容器，但是 QEMU 模拟有些坑，导致 dotnet 有些功能不能符合预期工作。且运行性能很低

现在我的需求是不能按照 [X86模拟龙芯与编译 .NET CoreCLR - 痴者工良 - 博客园](https://www.cnblogs.com/whuanle/p/13196860.html ) 博客提供的方法进行，此博客里面使用的是 MIPS 的，非龙架构的，不符合本文的需求。但依然感谢大佬帮忙踩坑

先准备好一台 x86_64 的 debian 12 的设备，接着按照通用的方法安装上 docker 和相关工具，推荐一切按照默认的来。我这里就因为担心磁盘空间问题，踩了一个坑，详细请看 [docker中无法使用sudo命令，提示没有root权限或者文件系统挂载没有nosuid选项_an nfs file system without root privileges?-CSDN博客](https://blog.csdn.net/xinwenfei/article/details/130158059 )

官方 docker 安装方法： <https://docs.docker.com/engine/install/debian/>

为了能够使用上龙芯旧世界的 QEMU 需要使用特别的版本，我的伙伴 [lsj](https://blog.sdlsj.net/ ) 告诉我说 QEMU 有内核和用户态的部分，不仅仅只是靠底层模拟龙架构指令，还需要一些用户态的支持，用户态部分的支持就和新旧世界 ABI1.0 和 ABI2.0 有关系

特别感谢 [lsj](https://blog.sdlsj.net/ ) 教会了我这个方法

从 [GitHub](https://github.com/lcpu-club/loong64-dockerfiles/issues/1#issuecomment-2109460316) 上找到了 [zhangguanzhang](https://github.com/zhangguanzhang) 大佬贴出来的龙芯官方的qemu提交人给的版本，以及 [msojocs](https://github.com/msojocs) 大佬的[测试](https://github.com/lcpu-club/loong64-dockerfiles/issues/1#issuecomment-2109327754)结果，即可知道，可以使用如下命令进行部署

```bash
$ sudo docker run --rm --privileged zhangguanzhang/qemu-user-static --reset -p yes
```

拉取 `zhangguanzhang/qemu-user-static` 需要到 docker io 拉取，如果大家拉不下来，可以发邮件向我要 zhangguanzhang_qemu-user-static.335cfbcdcc6d.tar 文件，我通过网盘给你

完成上述步骤之后，即可拉取旧世界的包，如以下命令

```bash
docker pull cr.loongnix.cn/library/debian:buster-slim
```

预期会提示以下警告

```
WARNING: The requested image's platform (linux/loong64) does not match the detected host platform (linux/amd64/v3) and no specific platform was requested
```

有以上警告是非常正确的事情，因为就是要拉取龙架构的，不匹配当前的 amd64 架构的

试试此时继续使用以下命令看能否进入到容器里

```bash
docker run -it cr.loongnix.cn/library/debian:buster-slim 
```

如能够进入到容器，再使用 `ls /lib64` 命令，如输出的是 `ld.so.1` 则证明确实是旧世界

```bash
root@ff641f574628:~# ls /lib64
ld.so.1
```

以上就是在 amd64 设备上，使用 QEMU 运行龙芯旧世界的 docker 的方法

我开始使用 <https://gitee.com/michael0066/qemu-loongarch64-static.git> 的方式做，但似乎这样做出来的是新世界的。在这里踩坑，感谢 [lsj](https://blog.sdlsj.net/ ) 的帮助，才让我找到正确的方法

尝试继续在里面打上 dotnet 的支持，如以下命令

```bash
root@ff641f574628:~# wget http://ftp.loongnix.cn/dotnet/8.0.7/8.0.7-1/pkg/dotnet-sdk-8.0.107-linux-loongarch64.tar.xz
root@ff641f574628:~# mkdir dotnet && tar -xvf dotnet-sdk-8.0.107-linux-loongarch64.tar.xz -C dotnet
root@ff641f574628:~# ln -s /root/dotnet/dotnet /usr/bin/dotnet
```

此时使用 `dotnet --info` 都是能正常输出的，证明 dotnet 能够最基础在此容器里运行

```bash
root@ff641f574628:~# dotnet --info
.NET SDK:
 Version:           8.0.107
 Commit:            1bdaef7265
 Workload version:  8.0.100-manifests.43c23f91

Runtime Environment:
 OS Name:     Loongnix
 OS Version:  20
 OS Platform: Linux
 RID:         linux-loongarch64
 Base Path:   /root/dotnet/sdk/8.0.107/

.NET workloads installed:
 Workload version: 8.0.100-manifests.43c23f91
There are no installed workloads to display.

Host:
  Version:      8.0.7
  Architecture: loongarch64
  Commit:       dbda949b78

.NET SDKs installed:
  8.0.107 [/root/dotnet/sdk]

.NET runtimes installed:
  Microsoft.AspNetCore.App 8.0.7 [/root/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.NETCore.App 8.0.7 [/root/dotnet/shared/Microsoft.NETCore.App]

Other architectures found:
  None

Environment variables:
  Not set

global.json file:
  Not found

Learn more:
  https://aka.ms/dotnet/info

Download .NET:
  http://www.loongnix.cn/zh/api/dotnet

Loongson's .NET version:
  8.0.107-1
```

然而当输入 `dotnet --help` 或 `dotnet nuget --version` 时，却有异常提示

```bash
root@ff641f574628:~# dotnet --help
System.IO.IOException: Not enough storage is available to process this command. : 'NuGet-Migrations'
   at System.Threading.Mutex.CreateMutexCore(Boolean initiallyOwned, String name, Boolean& createdNew)
   at System.Threading.Mutex..ctor(Boolean initiallyOwned, String name)
   at NuGet.Common.Migrations.MigrationRunner.Run(String migrationsDirectory)
   at NuGet.Common.Migrations.MigrationRunner.Run()
   at Microsoft.DotNet.Configurer.DotnetFirstTimeUseConfigurer.Configure()
   at Microsoft.DotNet.Cli.Program.ConfigureDotNetForFirstTimeUse(IFirstTimeUseNoticeSentinel firstTimeUseNoticeSentinel, IAspNetCertificateSentinel aspNetCertificateSentinel, IFileSentinel toolPathSentinel, Boolean isDotnetBeingInvokedFromNativeInstaller, DotnetFirstRunConfiguration dotnetFirstRunConfiguration, IEnvironmentProvider environmentProvider, Dictionary`2 performanceMeasurements)
   at Microsoft.DotNet.Cli.Program.ProcessArgs(String[] args, TimeSpan startupTime, ITelemetry telemetryClient)
   at Microsoft.DotNet.Cli.Program.Main(String[] args)
```

错误信息如下

```
System.IO.IOException: Not enough storage is available to process this command. : 'NuGet-Migrations'
   at System.Threading.Mutex.CreateMutexCore(Boolean initiallyOwned, String name, Boolean& createdNew)
   at System.Threading.Mutex..ctor(Boolean initiallyOwned, String name)
   at NuGet.Common.Migrations.MigrationRunner.Run(String migrationsDirectory)
   at NuGet.Common.Migrations.MigrationRunner.Run()
   at Microsoft.DotNet.Configurer.DotnetFirstTimeUseConfigurer.Configure()
   at Microsoft.DotNet.Cli.Program.ConfigureDotNetForFirstTimeUse(IFirstTimeUseNoticeSentinel firstTimeUseNoticeSentinel, IAspNetCertificateSentinel aspNetCertificateSentinel, IFileSentinel toolPathSentinel, Boolean isDotnetBeingInvokedFromNativeInstaller, DotnetFirstRunConfiguration dotnetFirstRunConfiguration, IEnvironmentProvider environmentProvider, Dictionary`2 performanceMeasurements)
   at Microsoft.DotNet.Cli.Program.ProcessArgs(String[] args, TimeSpan startupTime, ITelemetry telemetryClient)
   at Microsoft.DotNet.Cli.Program.Main(String[] args)
```

通过 [GitHub](https://github.com/dotnet/runtime/issues/80619#issuecomment-1464123143) 大佬的分析，可以定位到是 <https://github.com/NuGet/NuGet.Client/blob/3ab7d1462029e0b211a88040c952c9b8a679708d/src/NuGet.Core/NuGet.Common/Migrations/MigrationRunner.cs#LL23C17-L23C73> 这句代码抛出的。也看到 [Stephen Toub](https://github.com/stephentoub) 大佬在 [GitHub](https://github.com/dotnet/runtime/issues/80619#issuecomment-1458930876) 的如下内容分析

> Mutex on Linux needs to perform a variety of I/O operations, including creating a file on disk and creating a pthreads mutex; it's possible one of those is failing for some reason.
> 
> That error string looks like https://github.com/dotnet/runtime/blob/622277e02da5517aaeaa322be27079822b4d33e2/src/coreclr/pal/src/misc/errorstrings.cpp#L71, which is used in a variety of places.  For example, it'll be used in https://github.com/dotnet/runtime/blob/622277e02da5517aaeaa322be27079822b4d33e2/src/coreclr/pal/src/synchobj/mutex.cpp#L791 if the operation encounters an EPERM errno from pthread_mutex_init.

但是无论是以下哪个帖子的内容，都帮不到忙

- <https://github.com/dotnet/runtime/issues/80619>
- <https://github.com/termux/termux-packages/issues/516#issuecomment-2109348032>
- <https://github.com/NuGet/Home/issues/12365#issuecomment-1463016158>
- <https://github.com/dotnet/runtime/issues/91987#issuecomment-2236418465>
- <https://gist.github.com/pchalamet/7c17be4cb4e74c7cacf36f94bf62b4ea>

其中有人说可能是 root 权限的问题，于是我就在不断倒腾各种权限，但依然失败。第二天 [lsj](https://blog.sdlsj.net/ ) 告诉我说可能是 QEMU 哪些模拟不正确，让我试试在龙芯物理设备上跑跑看

于是我就在 debian 里面，使用 docker image save 保存到本地，然后传到龙芯物理设备上，使用 docker image load 命令加载起来

接着再在龙芯物理设备上使用 docker run -it 方式进入容器，尝试各种 dotnet 命令都没有问题。由于使用的是相同的 docker image 因此可以证明是在 debian 12 上使用 QEMU 模拟不正确导致的问题，而不是制作出来的 docker image 有问题或 dotnet 有问题

通过在龙芯物理设备上的对比，就可以了解到在 debian 12 上使用 QEMU 模拟不正确，导致了 dotnet 许多基础命令不可用

由于我使用的是 GitHub 上 [zhangguanzhang](https://github.com/zhangguanzhang) 大佬贴出来的龙芯官方的qemu提交人给的版本，既没有代码且我也不懂。于是我就放弃了走模拟这条路

## 参考文档

- [如何称呼龙架构？ - 咱龙了吗？](https://areweloongyet.com/docs/loong-or-loongarch/ )
- [旧世界与新世界 - 咱龙了吗？](https://areweloongyet.com/docs/old-and-new-worlds/ )
- [龙芯自主指令集LoongArch为什么是全新指令集，和MIPS不同在哪儿？- 辟谣 - 哔哩哔哩](https://www.bilibili.com/read/cv27388271/ )
- [龙芯云 https://loongson-cloud-community.github.io/Loongson-Cloud-Community/](https://loongson-cloud-community.github.io/Loongson-Cloud-Community/ )
- [容器镜像仓库 - 龙芯开源社区](https://www.loongnix.cn/zh/cloud/container-registry/ )
- [Loongnix操作系统源](https://pkg.loongnix.cn/loongnix/pool/main/)
- <https://hub.docker.com/u/loongarch64>
- [bash: xz: command not found - CSDN文库](https://wenku.csdn.net/answer/1s5dvi0vay )
- [Linux tar 命令解压tar.xz文件 - myfreax](https://www.myfreax.com/how-to-extract-unzip-tar-xz-file/ )
- [apt 与 apt-get — Linux 中软件包管理工具的区别 — AWS](https://aws.amazon.com/cn/compare/the-difference-between-apt-and-apt-get/ )
- [dotnet 基于 debian 创建一个 docker 的 sdk 镜像](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-debian-%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA-docker-%E7%9A%84-sdk-%E9%95%9C%E5%83%8F.html )
- [修复 Debian 安装 dotnet 失败 depends on ca-certificates](https://blog.lindexi.com/post/%E4%BF%AE%E5%A4%8D-Debian-%E5%AE%89%E8%A3%85-dotnet-%E5%A4%B1%E8%B4%A5-depends-on-ca-certificates.html )
- [制作一个能构建 dotnet AOT 的 gitlab ruuner 的 Debian docker 镜像](https://blog.lindexi.com/post/%E5%88%B6%E4%BD%9C%E4%B8%80%E4%B8%AA%E8%83%BD%E6%9E%84%E5%BB%BA-dotnet-AOT-%E7%9A%84-gitlab-ruuner-%E7%9A%84-Debian-docker-%E9%95%9C%E5%83%8F.html ) <!-- [制作一个能构建 dotnet AOT 的 gitlab runner 的 Debian docker 镜像 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18164886 ) -->
- [.net - A fatal error occurred. The folder [/usr/share/dotnet/host/fxr] does not exist - Stack Overflow](https://stackoverflow.com/questions/73753672/a-fatal-error-occurred-the-folder-usr-share-dotnet-host-fxr-does-not-exist )
- [Troubleshoot .NET package mix ups on Linux - .NET - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/install/linux-package-mixup?pivots=os-linux-redhat )
- [Globalization config settings - .NET - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/globalization )
- [X86模拟龙芯与编译 .NET CoreCLR - 痴者工良 - 博客园](https://www.cnblogs.com/whuanle/p/13196860.html )
- [dockerfile-新增用户并赋予sudo权限以及指定密码_dockerfile sudo-CSDN博客](https://blog.csdn.net/u010275850/article/details/120587850 )
- [【零基础入门Docker】Dockerfile中的USER指令以及dockerfile命令详解_docker_arthas777-云原生](https://devpress.csdn.net/cloudnative/66d57eb71328e17ef4733617.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MjcyNDU3NywiZXhwIjoxNzMwOTg0MzYzLCJpYXQiOjE3MzAzNzk1NjMsInVzZXJuYW1lIjoibGluZGV4aV9nZCJ9.8Hr4iU7YSi77wJixyDhn0b0COvfoda8nZ4q6V_76KGM )
- [Linux下创建普通用户遇到的问题及解决办法_安装linux忘记创建普通用户-CSDN博客](https://blog.csdn.net/weixin_42570192/article/details/132828091 )
- [linux给普通用户分配root权限_linux给多用户root权限-CSDN博客](https://blog.csdn.net/qq_20817327/article/details/115673697 )
- [linux给用户添加root权限方法总结-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1725832 )
- [docker容器以ROOT账号登录（获取ROOT权限/ROOT密码） - cn2024 - 博客园](https://www.cnblogs.com/smallfa/p/11141936.html )
- [docker中无法使用sudo命令，提示没有root权限或者文件系统挂载没有nosuid选项_an nfs file system without root privileges?-CSDN博客](https://blog.csdn.net/xinwenfei/article/details/130158059 )
- [sudo: effective uid is not 0, is /usr/bin/sudo on a file system with the 'nosuid' option set or an NFS file system without root privileges? · Issue #1100 · docker/for-linux](https://github.com/docker/for-linux/issues/1100 )
- [sudo: effective uid is not 0, is /usr/bin/sudo on a file system with the 'nosuid' option set or an NFS file system without root privileges? - CSDN文库](https://wenku.csdn.net/answer/5210b49775f449e78e659e295962bc12 )
- [[daily][centos][sudo] sudo 报错 - toong - 博客园](https://www.cnblogs.com/hugetong/p/7715570.html )
- <https://docs.docker.com/engine/install/debian/>
- <https://gitee.com/michael0066/qemu-loongarch64-static.git>
- [x86 架构运行 其他架构镜像-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1952902 )
- [Debian 12 / Ubuntu 24.04 安装 Docker 以及 Docker Compose 教程 - 烧饼博客](https://u.sb/debian-install-docker/ )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
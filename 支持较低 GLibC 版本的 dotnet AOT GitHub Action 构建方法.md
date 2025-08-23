# 支持较低 GLibC 版本的 dotnet AOT GitHub Action 构建方法

无论是麒麟还是 UOS 系统，所采用的 GLibC 版本都比较低。默认在 GitHub Action 上构建用的是 ubuntu-latest 较新的版本。进行 AOT 发布的 dotnet 程序将因为 GLibC 版本太新，而无法在麒麟或 UOS 等国产化系统上跑起来

<!--more-->
<!-- CreateTime:2025/08/23 07:18:20 -->

<!-- 发布 -->
<!-- 博客 -->

为了能够在带着较低的 GLibC 的麒麟或 UOS 等国产化系统跑起来经由 GitHub Action 构建的 dotnet AOT 应用，就要求在 GitHub Action 的发布环境需要采用足够旧的系统版本，当前 2025 最佳的为 Debian 10（buster）版本

本文将告诉大家应该如何编写 GitHub Action 的 yml 文件，从而能够在足够旧的 Debian 10（buster）进行 dotnet AOT 构建

开始之前先来看一下我手头的麒麟系统上的 glibc 的版本

```
$ ldd --version
ldd (Ubuntu GLIBC 2.31-0kylin9.1k21.6) 2.31
```

以及 UOS 系统的 glibc 的版本

```
$ ldd --version
ldd (Debian GLIBC 2.28.30-deepin1) 2.28
```

在 GitHub Action 的基础设施机制里面，提供了内置在 docker 容器镜像里面运行的方式，这将大大方便咱的工作

在 yml 文件里面，通过指定 container 的方式配置采用 `debian:buster-slim` 容器来运行，其示例代码如下

```yml
  PackOnLinuxX64:

    runs-on: ubuntu-latest
    container:
      image: debian:buster-slim
```

由于 `debian:buster-slim` 容器里面啥都没有，直接在里面跑 actions/setup-dotnet 将会失败，大概的提示如下

```
Run actions/setup-dotnet@v4
/usr/bin/docker exec  03691b621c91f406e8b9ff39659f0ee58fe971995fd7ec9eef52f0dea518469c sh -c "cat /etc/*release | grep ^ID"
/__w/_actions/actions/setup-dotnet/v4/externals/install-dotnet.sh --skip-non-versioned-files --runtime dotnet --channel LTS
dotnet_install: Error: curl (recommended) or wget are required to download dotnet. Install missing prerequisite to proceed.
Warning: Failed to install dotnet runtime + cli, exit code: 1. dotnet_install: Error: curl (recommended) or wget are required to download dotnet. Install missing prerequisite to proceed.

/__w/_actions/actions/setup-dotnet/v4/externals/install-dotnet.sh --skip-non-versioned-files --channel 9.0
dotnet_install: Error: curl (recommended) or wget are required to download dotnet. Install missing prerequisite to proceed.
Error: Failed to install dotnet, exit code: 1. dotnet_install: Error: curl (recommended) or wget are required to download dotnet. Install missing prerequisite to proceed.
```

解决此问题的方法就是安装工具，继续添加如下代码进行工具的安装

```yml
  PackOnLinuxX64:

    runs-on: ubuntu-latest
    container:
      image: debian:buster-slim

    steps:
    - name: InstallTool
      run: |
          dpkg --add-architecture arm64
          apt-get clean
          apt-get update
          apt-get install libicu-dev -y
          apt-get install libssl-dev -y
          apt-get install wget -y
          apt-get install curl -y
          apt-get install clang llvm -y
          apt-get install gcc-aarch64-linux-gnu -y
          apt-get install binutils-aarch64-linux-gnu -y
          apt-get install zlib1g-dev -y
          apt-get install zlib1g-dev:arm64 -y
```

然而在以上安装过程中，将会看到如下错误

```
Ign:1 http://deb.debian.org/debian buster InRelease
Ign:2 http://deb.debian.org/debian-security buster/updates InRelease
Ign:3 http://deb.debian.org/debian buster-updates InRelease
Err:4 http://deb.debian.org/debian buster Release
  404  Not Found [IP: 146.75.30.132 80]
Err:5 http://deb.debian.org/debian-security buster/updates Release
  404  Not Found [IP: 146.75.30.132 80]
Err:6 http://deb.debian.org/debian buster-updates Release
  404  Not Found [IP: 146.75.30.132 80]
Reading package lists...
E: The repository 'http://deb.debian.org/debian buster Release' does not have a Release file.
E: The repository 'http://deb.debian.org/debian-security buster/updates Release' does not have a Release file.
E: The repository 'http://deb.debian.org/debian buster-updates Release' does not have a Release file.
```

这个错误的原因是 Debian 10 (buster) 已经停止支持，被移动到 archive 里，需要更换包源。我在另一篇博客详细介绍了其原因，详细请参阅：[制作一个能构建 dotnet AOT 的 gitlab ruuner 的 Debian docker 镜像](https://blog.lindexi.com/post/%E5%88%B6%E4%BD%9C%E4%B8%80%E4%B8%AA%E8%83%BD%E6%9E%84%E5%BB%BA-dotnet-AOT-%E7%9A%84-gitlab-ruuner-%E7%9A%84-Debian-docker-%E9%95%9C%E5%83%8F.html )
<!-- [制作一个能构建 dotnet AOT 的 gitlab runner 的 Debian docker 镜像 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18164886 ) -->

本文这里简单起见，直接修改包源，添加如下代码进行修改


```yml
    steps:
      # 由于 Debian 10 (buster) 停止维护了，需要换成 archive.debian.org 源
    - name: UpdateSource
      run: |
          rm /etc/apt/sources.list
          echo 'deb http://archive.debian.org/debian buster main contrib non-free'  >> /etc/apt/sources.list
          echo 'deb http://archive.debian.org/debian buster-updates main contrib non-free'  >> /etc/apt/sources.list
          echo 'deb http://archive.debian.org/debian-security buster/updates main contrib non-free'  >> /etc/apt/sources.list
```

这里需要说明的是，除了 `http://archive.debian.org/debian` 之外，后面两个也是非常重要的，否则将会遇到找不到 libc-dev 而失败，其失败提示如下

```
Get:1 http://archive.debian.org/debian buster InRelease [122 kB]
Get:2 http://archive.debian.org/debian buster/main amd64 Packages [7909 kB]
Get:3 http://archive.debian.org/debian buster/contrib arm64 Packages [38.4 kB]
Get:4 http://archive.debian.org/debian buster/non-free arm64 Packages [53.9 kB]
Get:5 http://archive.debian.org/debian buster/contrib amd64 Packages [50.1 kB]
Get:6 http://archive.debian.org/debian buster/non-free amd64 Packages [87.8 kB]
Get:7 http://archive.debian.org/debian buster/main arm64 Packages [7737 kB]
Fetched 16.0 MB in 2s (8464 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 libicu-dev : Depends: libc6-dev but it is not going to be installed or
                       libc-dev
E: Unable to correct problems, you have held broken packages.
```

由于 GitHub Action 是跑在国外的，就没有必要去使用腾讯的源，直接使用 archive.debian.org 就可以了

完成以上步骤之后，即可按照正常方式进行安装 .NET SDK 了，代码如下

```yml
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: |
          3.1.x
          5.0.x
          6.0.x
          9.0.x
```

如果后续步骤需要使用 dotnet tool 的话，需要额外添加以下步骤进行设置，代码如下

```yml
    - name: Add .NET global tools to PATH
      run: echo "$HOME/.dotnet/tools" >> $GITHUB_PATH
```

如果没有以上设置，则会在后续安装工具的时候给出警告，如以下步骤的安装 dotnet campus 的 TagToVersion 工具的例子

```yml
    - name: Install dotnet tool
      run: dotnet tool install -g dotnetCampus.TagToVersion
```

安装过程中会看到如下提示

```
Tools directory '/github/home/.dotnet/tools' is not currently on the PATH environment variable.
If you are using bash, you can add it to your profile by running the following command:

cat << \EOF >> ~/.bash_profile
# Add .NET Core SDK tools
export PATH="$PATH:/github/home/.dotnet/tools"
EOF

You can add it to the current session by running the following command:

export PATH="$PATH:/github/home/.dotnet/tools"

You can invoke the tool using the following command: dotnet-TagToVersion
Tool 'dotnetcampus.tagtoversion' (version '1.0.11') was successfully installed.
```

此时使用 `export PATH="$PATH:/github/home/.dotnet/tools"` 是无效的，会在 GitHub Action 里提示找不到 dotnet 工具。正确做法是需要添加到 GITHUB_PATH 里，采用的是 `echo "$HOME/.dotnet/tools" >> $GITHUB_PATH` 命令，详细请参阅 <https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands>

完成以上步骤之后，就可以正常打包了

在本文开始的安装工具步骤里，添加了 linux-arm64 的交叉构建，以上步骤即可构建出 linux-x64 和 linux-arm64 的 dotnet AOT 应用

以下是本文的示例文件代码，此文件是 <https://github.com/dotnet-campus/DocumentFormat.OpenXml.Extensions> 项目的构建脚本

以下代码内容有所删减，如对整个构建脚本感兴趣，还请自行到 <https://github.com/dotnet-campus/DocumentFormat.OpenXml.Extensions> 项目获取完全的构建脚本

```yml
name: Publish MediaConverters NuGet

on: 
  push:
    tags:
    - '*'

jobs:

  PackOnLinuxX64:

    runs-on: ubuntu-latest
    container:
      image: debian:buster-slim

    steps:
      # 由于 Debian 10 (buster) 停止维护了，需要换成 archive.debian.org 源头
    - name: UpdateSource
      run: |
          rm /etc/apt/sources.list
          echo 'deb http://archive.debian.org/debian buster main contrib non-free'  >> /etc/apt/sources.list
          echo 'deb http://archive.debian.org/debian buster-updates main contrib non-free'  >> /etc/apt/sources.list
          echo 'deb http://archive.debian.org/debian-security buster/updates main contrib non-free'  >> /etc/apt/sources.list
    - name: InstallTool
      run: |
          dpkg --add-architecture arm64
          apt-get clean
          apt-get update
          apt-get install libicu-dev -y
          apt-get install libssl-dev -y
          apt-get install wget -y
          apt-get install curl -y
          apt-get install clang llvm -y
          apt-get install gcc-aarch64-linux-gnu -y
          apt-get install binutils-aarch64-linux-gnu -y
          apt-get install zlib1g-dev -y
          apt-get install zlib1g-dev:arm64 -y

    - uses: actions/checkout@v4
 
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: |
          3.1.x
          5.0.x
          6.0.x
          9.0.x
 
    - name: Add .NET global tools to PATH
      run: echo "$HOME/.dotnet/tools" >> $GITHUB_PATH

    - name: Install dotnet tool
      run: dotnet tool install -g dotnetCampus.TagToVersion

    - name: Set tag to version  
      run: dotnet-TagToVersion -t 1.0.0
 
    - name: Build with dotnet
      run: dotnet build --configuration Release src/MediaConverters/MediaConverters.sln

    - name: Publish and Pack linux-x64
      run: |
        dotnet publish -c Release -r linux-x64 src/MediaConverters/MediaConverters.Tool/MediaConverters.Tool.csproj
        dotnet pack --configuration Release /p:RuntimeIdentifier=linux-x64 src/MediaConverters/MediaConverters.Tool.RuntimeNuGet/MediaConverters.Tool.RuntimeNuGet.csproj
```
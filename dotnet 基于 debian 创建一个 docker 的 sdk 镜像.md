# dotnet 基于 debian 创建一个 docker 的 sdk 镜像

我不能用官方的镜像，因为我需要在小伙伴构建的 debian 镜像上面安装 dotnet sdk 用来做构建，其实在 docker 里面需要找到一个个文件，然后复制代码

<!--more-->
<!-- CreateTime:2020/3/8 11:33:47 -->



这是小伙伴做的一个 jenkins 的镜像 jenkins/slave 我需要在这个镜像基础上安装 dotnet 进行构建

```csharp
FROM jenkins/slave
```

在安装 dotnet 之前需要通过官方 https://github.com/dotnet/dotnet-docker 找到对应的 docker 文件，这样可以进行复制，可以看到官方是引用下面代码 buildpack-deps:buster-scm 这个库

```csharp
FROM buildpack-deps:buster-scm 
```

需要找到 buildpack-deps:buster-scm 的代码

找到这个库的代码 https://github.com/docker-library/buildpack-deps/blob/b0fc01aa5e3aed6820d8fed6f3301e0542fbeb36/buster/curl/Dockerfile 可以复制，但是这个库依赖 FROM buildpack-deps:buster-curl 这个库，通过 docker hub 找到 https://github.com/docker-library/buildpack-deps/blob/b0fc01aa5e3aed6820d8fed6f3301e0542fbeb36/buster/curl/Dockerfile 也可以复制

```
# https://github.com/docker-library/buildpack-deps/blob/b0fc01aa5e3aed6820d8fed6f3301e0542fbeb36/buster/curl/Dockerfile
# FROM debian:buster
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        netbase \
        wget \
    && rm -rf /var/lib/apt/lists/*
 
RUN set -ex; \
    if ! command -v gpg > /dev/null; then \
        apt-get update; \
        apt-get install -y --no-install-recommends \
            gnupg \
            dirmngr \
        ; \
        rm -rf /var/lib/apt/lists/*; \
    fi
 
# FROM buildpack-deps:buster-curl
# https://github.com/docker-library/buildpack-deps/blob/99a1c33fda559272e9322b02a5d778bbd04154e7/buster/scm/Dockerfile
# procps is very common in build systems, and is a reasonably small package
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        mercurial \
        openssh-client \
        subversion \
        \
        procps \
    && rm -rf /var/lib/apt/lists/*
```

使用 dotnet 官方地址下载速度太慢了，我请了小伙伴帮我下载，然后我将 dotnet-sdk-3.1.102-linux-x64.tar.gz 和 PowerShell.Linux.x64.7.0.0-rc.2.nupkg 放在 docker 文件相同文件夹

接下来是复制 dotnet 的代码

```
# https://raw.githubusercontent.com/dotnet/dotnet-docker/74c92451ecbd2876280ad51736a6eea4e98a1fb2/3.1/sdk/buster/amd64/Dockerfile
# FROM buildpack-deps:buster-scm
ENV \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip \
    # PowerShell telemetry for docker image usage
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetCoreSDK-Debian-10
 
# Install .NET CLI dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libc6 \
        libgcc1 \
        libgssapi-krb5-2 \
        libicu63 \
        libssl1.1 \
        libstdc++6 \
        zlib1g \
    && rm -rf /var/lib/apt/lists/*
 
# Install .NET Core SDK
# 不从 https://dotnetcli.azureedge.net 下载，从本地复制
RUN dotnet_sdk_version=3.1.102 \
#    && curl -SL --output dotnet.tar.gz https://dotnetcli.azureedge.net/dotnet/Sdk/$dotnet_sdk_version/dotnet-sdk-$dotnet_sdk_version-linux-x64.tar.gz \
    && dotnet_sha512='9cacdc9700468a915e6fa51a3e5539b3519dd35b13e7f9d6c4dd0077e298baac0e50ad1880181df6781ef1dc64a232e9f78ad8e4494022987d12812c4ca15f29' \
    && echo "$dotnet_sha512 dotnet.tar.gz" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet \
    && tar -ozxf dotnet.tar.gz -C /usr/share/dotnet \
    && rm dotnet.tar.gz \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
    # Trigger first run experience by running arbitrary cmd
    && dotnet help
 
# 不从 https://pwshtool.blob.core.windows.net 下载，从本地复制
# Install PowerShell global tool
RUN powershell_version=7.0.0-rc.2 \
#    && curl -SL --output PowerShell.Linux.x64.$powershell_version.nupkg https://pwshtool.blob.core.windows.net/tool/$powershell_version/PowerShell.Linux.x64.$powershell_version.nupkg \
#   && powershell_sha512='59abcc11bd43fc8c1938a1854447c762092f03b5e2c6c354a82559eed6852e3920c5543c085fbe6fbe98871f96cd7409bb76b1537d3d8dee4e7432d578ec7603' \
#   && echo "$powershell_sha512  PowerShell.Linux.x64.$powershell_version.nupkg" | sha512sum -c - \
    && mkdir -p /usr/share/powershell \
    && dotnet tool install --add-source / --tool-path /usr/share/powershell --version $powershell_version PowerShell.Linux.x64 \
    && dotnet nuget locals all --clear \
    && rm /PowerShell.Linux.x64.$powershell_version.nupkg \
    && ln -s /usr/share/powershell/pwsh /usr/bin/pwsh \
    && chmod 755 /usr/share/powershell/pwsh \
    # To reduce image size, remove the copy nupkg that nuget keeps.
```

运行 `docker build -t dotnet-sdk:3.0.102 .` 就可以创建了

全部代码

```
FROM jenkins/slave

COPY dotnet-sdk-3.1.102-linux-x64.tar.gz dotnet.tar.gz
COPY PowerShell.Linux.x64.7.0.0-rc.2.nupkg /

# 使用国内源
RUN echo "" >> /etc/apt/sources.list \
    && echo "deb http://ftp.cn.debian.org/debian/ buster main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb-src http://ftp.cn.debian.org/debian/ buster main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb http://ftp.cn.debian.org/debian-security/ buster/updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb-src http://ftp.cn.debian.org/debian-security/ buster/updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb http://ftp.cn.debian.org/debian/ buster-updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb-src http://ftp.cn.debian.org/debian/ buster-updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb http://ftp.cn.debian.org/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb-src http://ftp.cn.debian.org/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list

# https://github.com/docker-library/buildpack-deps/blob/b0fc01aa5e3aed6820d8fed6f3301e0542fbeb36/buster/curl/Dockerfile
# FROM debian:buster
RUN apt-get update && apt-get install -y --no-install-recommends \
		ca-certificates \
		curl \
		netbase \
		wget \
	&& rm -rf /var/lib/apt/lists/*

RUN set -ex; \
	if ! command -v gpg > /dev/null; then \
		apt-get update; \
		apt-get install -y --no-install-recommends \
			gnupg \
			dirmngr \
		; \
		rm -rf /var/lib/apt/lists/*; \
	fi

# FROM buildpack-deps:buster-curl
# https://github.com/docker-library/buildpack-deps/blob/99a1c33fda559272e9322b02a5d778bbd04154e7/buster/scm/Dockerfile
# procps is very common in build systems, and is a reasonably small package
RUN apt-get update && apt-get install -y --no-install-recommends \
		git \
		mercurial \
		openssh-client \
		subversion \
		\
		procps \
	&& rm -rf /var/lib/apt/lists/*

# https://raw.githubusercontent.com/dotnet/dotnet-docker/74c92451ecbd2876280ad51736a6eea4e98a1fb2/3.1/sdk/buster/amd64/Dockerfile
# FROM buildpack-deps:buster-scm
ENV \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip \
    # PowerShell telemetry for docker image usage
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetCoreSDK-Debian-10

# Install .NET CLI dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libc6 \
        libgcc1 \
        libgssapi-krb5-2 \
        libicu63 \
        libssl1.1 \
        libstdc++6 \
        zlib1g \
    && rm -rf /var/lib/apt/lists/*

# Install .NET Core SDK
# 不从 https://dotnetcli.azureedge.net 下载，从本地复制
RUN dotnet_sdk_version=3.1.102 \
#    && curl -SL --output dotnet.tar.gz https://dotnetcli.azureedge.net/dotnet/Sdk/$dotnet_sdk_version/dotnet-sdk-$dotnet_sdk_version-linux-x64.tar.gz \
    && dotnet_sha512='9cacdc9700468a915e6fa51a3e5539b3519dd35b13e7f9d6c4dd0077e298baac0e50ad1880181df6781ef1dc64a232e9f78ad8e4494022987d12812c4ca15f29' \
    && echo "$dotnet_sha512 dotnet.tar.gz" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet \
    && tar -ozxf dotnet.tar.gz -C /usr/share/dotnet \
    && rm dotnet.tar.gz \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
    # Trigger first run experience by running arbitrary cmd
    && dotnet help

# 不从 https://pwshtool.blob.core.windows.net 下载，从本地复制
# Install PowerShell global tool
RUN powershell_version=7.0.0-rc.2 \
#    && curl -SL --output PowerShell.Linux.x64.$powershell_version.nupkg https://pwshtool.blob.core.windows.net/tool/$powershell_version/PowerShell.Linux.x64.$powershell_version.nupkg \
#   && powershell_sha512='59abcc11bd43fc8c1938a1854447c762092f03b5e2c6c354a82559eed6852e3920c5543c085fbe6fbe98871f96cd7409bb76b1537d3d8dee4e7432d578ec7603' \
#   && echo "$powershell_sha512  PowerShell.Linux.x64.$powershell_version.nupkg" | sha512sum -c - \
    && mkdir -p /usr/share/powershell \
    && dotnet tool install --add-source / --tool-path /usr/share/powershell --version $powershell_version PowerShell.Linux.x64 \
    && dotnet nuget locals all --clear \
    && rm /PowerShell.Linux.x64.$powershell_version.nupkg \
    && ln -s /usr/share/powershell/pwsh /usr/bin/pwsh \
    && chmod 755 /usr/share/powershell/pwsh \
    # To reduce image size, remove the copy nupkg that nuget keeps.

```

我将本文用到两个资源 CSDN 如果没法下载请发我邮件

[dotnet-sdk-3.1.102-linux-x64.tar.gz](https://download.csdn.net/download/lindexi_gd/12233549)

[PowerShell.Linux.x64.7.0.0-rc.2.nupkg ](https://download.csdn.net/download/lindexi_gd/12233550)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

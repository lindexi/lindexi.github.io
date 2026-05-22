# Windows 上高效开发与调试 docker 容器内 dotnet 程序的方案

在 Windows 上开发需要在容器内运行的 .NET 程序时，一个很烦人的问题就是：每次改完代码，怎么快速把编译产物弄进容器里？

本文将介绍一种无需任何第三方工具的方法：利用 WSL 网络驱动器映射，配合 `dotnet publish` 命令直接输出到容器目录，实现 发布即就绪 的体验。再结合 SSH 远程调试，整体的开发效率会有明显提升。

<!--more-->
<!-- CreateTime:2026/05/16 07:20:11 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 痛点：从宿主机到容器的文件同步

开发容器内运行的程序时，典型的调试循环是这样的：

1. 在宿主机上改代码。
2. 编译或发布。
3. 把产物弄进容器里。
4. 在容器内运行，观察效果。

其中第三步最令人头疼。一般来说有两种常见方案：

- **第三方同步工具**：比如 SyncTool 之类的文件同步软件，在宿主机和容器之间搭一座桥。这需要额外安装和配置工具，同步延迟也不可控。
- **挂载 SMB 共享**：把宿主机的某个目录通过 SMB 协议共享给容器。配置流程长，涉及用户权限、防火墙等，容易踩坑。

有没有办法，一条命令下去，产物直接就出现在容器里？答案是有的。

## 核心思路：利用 WSL 网络驱动器映射

在 Windows 上使用 Docker Desktop 需要满足其商业许可要求，对于一些团队来说这可能是个麻烦事。而 Podman 是一个完全开源、无需守护进程的容器工具，其命令行接口与 Docker 高度兼容，基本上只需要将 `docker` 命令替换为 `podman` 即可完成迁移。本文将集中地采用 podman 来作为容器工具

在 Windows 上使用 Podman 时，容器实际上是跑在 WSL 虚拟机里面的。当使用 Podman 的数据卷挂载功能时，数据卷对应的实际路径就在 WSL 的文件系统中。

根据 [在 windows 上运行的 podman 默认的挂载相对路径是什么](https://www.cnblogs.com/lindexi/p/18156725) 以及 [Windows10内置Linux子系统（WSL）映射本地盘符](https://www.cnblogs.com/RainFate/p/16771349.html) 两篇文档可以知道，Podman 通过相对路径挂载的数据卷，实际存储在 WSL 的以下位置：

```
~/.local/share/containers/storage/volumes/
```

而 Windows 提供了将 WSL 路径映射为网络驱动器的能力，映射地址为 `\\wsl.localhost\podman-machine-default`。

一旦映射完成，WSL 里的路径就可以直接在 Windows 上用盘符访问。此时，`dotnet publish` 的输出路径直接指向这个盘符下的对应目录，产物就一步到位进入了容器可见的数据卷。

整个链路如下图：

```
dotnet publish → N:\...\volumes\app\_data → WSL 数据卷 → 容器内 /app
```

没有任何中间环节，没有同步等待。

## 启动容器并挂载数据卷

首先用 Podman 启动一个带数据卷的容器。这里以 `ubuntu:24.04` 作为基础镜像，同时映射好后续 SSH 调试需要的端口：

```
podman run -p 8080:8080 -p 5022:22 -v app:/app -it ubuntu:24.04 /bin/bash
```

各参数的作用如下：

1. `-p 8080:8080` 和 `-p 5022:22`：将容器内的 Web 端口和 SSH 端口分别映射到宿主机，为后续调试做准备。
2. `-v app:/app`：将名为 `app` 的数据卷挂载到容器内的 `/app` 目录。这是宿主机和容器之间文件共享的关键通道。
3. `-it ubuntu:24.04 /bin/bash`：以交互模式启动 Ubuntu 24.04 容器，并获得一个可用的 shell。

这里的数据卷 `app` 在 WSL 中的实际路径就是：

```
~/.local/share/containers/storage/volumes/app/_data
```

这个 `_data` 目录就是数据卷的根。往里面写什么，容器内 `/app` 就能看到什么。

## 映射 WSL 路径为 Windows 网络驱动器

接下来要把 WSL 路径暴露到 Windows 下。操作步骤如下：

1. 打开 Windows 资源管理器，点击"此电脑"。
2. 在顶部菜单栏选择"映射网络驱动器"。
3. 选择一个未使用的驱动器号，例如 `N:`。
4. 在文件夹输入框中填入 `\\wsl.localhost\podman-machine-default`。
5. 点击完成。

注意，这里的 `podman-machine-default` 是 Podman 默认创建的 WSL 虚拟机名称。如果你曾经自定义过虚拟机名称，需要替换为实际名称，可以通过 `podman machine list` 查看。

映射成功后，WSL 中的 `/home/user/.local/share/containers/storage/volumes/app/_data` 在 Windows 上对应的路径就是：

```
N:\home\user\.local\share\containers\storage\volumes\app\_data
```

## 一步发布：dotnet publish 直接输出到容器目录

有了网络驱动器映射，接下来就是核心操作了。在 Windows 上执行以下命令：

```
dotnet publish -c release -r linux-x64 --tl:off --sc -o N:\home\user\.local\share\containers\storage\volumes\app\_data
```

逐段解释：

1. `-c release`：使用 Release 配置，生成优化后的产物。
2. `-r linux-x64`：目标运行时指定为 linux-x64，因为容器内是 Linux 环境。
3. `--tl:off`：关闭终端日志器（Terminal Logger），避免输出路径相关的干扰信息。
4. `--sc`（即 `--self-contained`）：发布为自包含应用。这样容器内不需要额外安装 .NET 运行时，直接就能跑。
5. `-o N:\...\volumes\app\_data`：输出目录直接指向映射后的 WSL 数据卷路径。

命令执行完，产物就已经在容器内的 `/app` 目录里了。切换到容器终端，直接 `dotnet /app/YourApp.dll` 就能跑起来或 `./YourApp` 命令直接启动

注： 可能需要先 `chmod +x YourApp` 添加权限

相比传统方案，这里完全没有"同步"这个步骤。`dotnet publish` 结束就等于部署完成，零等待。

## 开启 SSH 远程调试

程序能在容器内运行只是第一步，还需要能调试。

SSH 调试的配置也很直接。首先，默认容器内的 root 用户没有密码，需要先设置一个：

```
passwd root
```

按提示输入密码，比如 `123`。

由于启动容器时已经通过 `-p 5022:22` 把容器的 22 端口映射到了宿主机的 5022 端口，现在就可以通过以下信息连接到容器：

- 主机：`127.0.0.1`
- 端口：`5022`
- 用户名：`root`
- 密码：刚才设置的那个

验证连接是否通畅：

```
ssh root@127.0.0.1 -p 5022
```

连接成功后，在 VisualStudio 中配置"附加到进程"，连接类型选择 SSH，填入上述连接信息，就能附加到容器内运行的 dotnet 进程进行断点调试了。这里的详细步骤请参阅： [UOS 开启 VisualStudio 远程调试 .NET 应用之旅](https://blog.lindexi.com/post/UOS-%E5%BC%80%E5%90%AF-VisualStudio-%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95-.NET-%E5%BA%94%E7%94%A8%E4%B9%8B%E6%97%85.html )
<!-- [UOS 开启 VisualStudio 远程调试 .NET 应用之旅 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18086533 ) -->

## 总结

整个方案的核心在于：把 WSL 数据卷映射为 Windows 网络驱动器，让 `dotnet publish` 的输出路径直接指向容器可见的目录。这样一来，发布和部署合二为一，省掉了同步这个步骤。

步骤回顾：

1. `podman run` 启动容器，用 `-v` 挂载数据卷。
2. 将 `\\wsl.localhost\podman-machine-default` 映射为 Windows 网络驱动器。
3. `dotnet publish -o` 直接输出到网络驱动器对应的数据卷路径。
4. 容器内 `passwd root` 设置密码，开启 SSH。
5. VisualStudio 通过 SSH 附加到容器进程进行调试。

整个过程不需要任何第三方同步工具，也不需要配置 SMB。改完代码，一条发布命令，产物就在容器里了。

## 参考文档

- [在 windows 上运行的 podman 默认的挂载相对路径是什么](https://www.cnblogs.com/lindexi/p/18156725)
- [Windows10内置Linux子系统（WSL）映射本地盘符](https://www.cnblogs.com/RainFate/p/16771349.html)
- [UOS 开启 VisualStudio 远程调试 .NET 应用之旅](https://blog.lindexi.com/post/UOS-%E5%BC%80%E5%90%AF-VisualStudio-%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95-.NET-%E5%BA%94%E7%94%A8%E4%B9%8B%E6%97%85.html )
<!-- [UOS 开启 VisualStudio 远程调试 .NET 应用之旅 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18086533 ) -->

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html)
---
title: UOS 开启 VisualStudio 远程调试 .NET 应用之旅
description: 本文记录的是在 Windows 系统里面，使用 VisualStudio 2022 远程调试运行在 UOS 里面 dotnet 应用的配置方法
tags: VisualStudio
category: 
---

<!-- CreateTime:2024/03/21 07:05:36 -->

<!-- 发布 -->
<!-- 博客 -->

本文写于 2024.03.19 如果你阅读本文的时间距离本文编写的时间过于长，那本文可能包含过期的知识

我将以我的 UOS 虚拟机作为例子告诉大家如何在 Windows 系统里面，使用 VisualStudio 2022 远程调试运行在 UOS 里面 dotnet 应用。这里的 dotnet 应用不仅包含纯控制台即可实现的 ASP.NET Core 也包括带 GUI 的程序，包括 CPF 、 UNO/MAUI、 Avalonia 等 UI 框架上构建的可运行在 UOS Linux 系统上的应用

## 准备

第一步是保证 UOS 和 Windows 构成局域网网络，或可直接联通的网络。这一步可自行了解网络配置，只要双方网络能互通即可，挂在一个局域网内相同网段是最好的选择

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203530214-1831118243.png)

其次是在 UOS 里面开启开发者模式，开启方法请参阅 [如何打开开发者模式？-UOS官方文档](https://doc.chinauos.com/content/poVHjoEBplouMytzM6i8)

准备工作就此两步，接下来就是进行配置

## 配置

接着的配置步骤可以分为两大块，分别是 SSH 的开启以及 VisualStudio 的连接

### 开启 SSH 的支持

默认的 UOS 是没有开启 SSH 的支持的，本文将使用 Tame-complexity 大佬在博客园里面的写的 [Linux开启ssh - Tame-complexity - 博客园](https://www.cnblogs.com/linshengqian/p/15065571.html ) 博客里面的部分方法开启 ssh 连接。如果只是想完成 UOS 系统下的 SSH 配置，那么按照本文的步骤即可，除非遇到问题否则不需要现在就去阅读 Tame-complexity 大佬的博客，防止大家只阅读了一半执行了错误的命令。本文将会给出大家具体且完全的步骤，具体步骤如下

先修改sshd服务配置文件，修改此文件需要权限。以下命令是在 UOS 的终端里面输入的命令行代码，在 UOS 开启终端可使用 `ctrl+alt+T` 快捷键。这里的 UOS 的终端就和 Windows 的 cmd 命令行相似

```
sudo su
> 输入你的密码
vi /etc/ssh/sshd_config
```

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅1.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203530617-1590035949.png)

进入 `sshd_config` 编辑入以下代码，这里有一个可简化的输入方法，那就是一般此文件里面只是包含大量被注释掉的配置，只需要找到对应的配置，解开注释即可。解开注释就是去掉行首的 `#` 字符，在这份配置文档里面，使用 `#` 作为表示此行是注释的用途

```
Port 22
ListenAddress 0.0.0.0
ListenAddress ::

PermitRootLogin yes
PasswordAuthentication yes
```

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅2.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203530974-610772611.png)

相信阅读到这里的伙伴是知道 vim 的基础操作的，如果不了解的话，还请自行查阅一下 vim 如何编辑文件以及如何保存退出哈。简单说就是按下 `i` 键进入编辑模式，通过上下左右方向键定位光标，通过删除和输入字符进行编辑。完成编辑之后，使用 `esc` 键退出编辑模式，再输入 `:wq` 即可完成保存退出

如果是解开注释而不是从零写入的话，需要小心 PermitRootLogin 配置项默认是 prohibit-password 的值，需要改为 yes 才对

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅3.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203531329-85894114.png)

配置完成之后，使用以下命令了解一下 ssh 的运行情况

```
/etc/init.d/ssh status
```

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅5.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203531683-1746556374.png)

默认新装的 UOS 都是关闭的状态，请使用以下代码进行开启

```
/etc/init.d/ssh start
```

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅6.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203532046-140968231.png)

开启之后，继续使用 `/etc/init.d/ssh status` 命令了解一下状态，预期是能够正确开启。如果不能正确开启，再根据错误信息，自行解决

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅7.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203532395-1657819035.png)

### 使用 VisualStudio 连接

打开一个 VisualStudio 2022 用于尝试附加调试连接

点击调试里面的附加进程

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅8.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203532767-32298849.png)

点击连接类型，切换到 ssh 类型

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅9.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203533083-1560022249.png)

在连接目标里面输入 UOS 的网络地址，这里支持 IP 或域名，如果开启的 SSH 端口非 22 端口，这里也可以不用输入端口，可以在后续界面再输入。相信如何查看 UOS 的网络的 IP 地址你已经学会了，如果还不知道的话，请自行搜寻答案。输入完成之后按回车，或者点查找按钮，但查找按钮有时候会不工作，推荐还是按回车好

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅10.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203533451-27117751.png)

按下回车之后，可以看到如下界面，输入你的端口和用户名密码，即可点击连接

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅11.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203533805-488744148.png)

如果能够看到提示是否要继续连接，请点击 是 按钮

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅12.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203534188-1519498816.png)

预期是能够看到以下调试界面，如能看到则表示连接成功，如果连接失败，请回到上一步，确定自己配置正确了 SSH 连接

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅13.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203534529-965668480.png)

常见的错误就是网络连不通，以及输入错账号密码等。比如说虚拟机进行了诡异的网络配置，导致物理机无法访问到虚拟机的网络等，这些就请大家自行解决网络问题

## 开启调试

接下来将使用 dotnet-campus 开源的下载器项目作为调试的例子

先从可用进程选中将要调试的应用，接着的细节是点击附加到里面，选择手动以及选择 托管(.NET Core for Unix)代码 选项。这是因为默认的 VisualStudio 的自动选择经常不工作

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅14.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203534897-943693501.png)

点击调试以下代码类型，然后选择 托管(.NET Core for Unix)代码 即可

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅15.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203535234-1240054273.png)

如果大家看到附加按钮是禁用状态，也可以尝试以上的选择 托管(.NET Core for Unix)代码 步骤

完成配置之后，即可点击附加按钮，基本都能成功

但如果有遇到以下失败的，提示 未能启动调试适配器，可在输出窗口查看额外的信息

> Unable to find debugger script at 'home/lin/.vs-debugger'.

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅16.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203535616-344676020.png)

看到错误有以下代码

```
[ERROR] 灾难性故障 (异常来自 HRESULT:0x8000FFFF (E_UNEXPECTED))
```

也有类似如下错误输出信息

```
11:34:39:958	未能启动调试适配器“coreclr”。
11:34:39:958	Unable to find debugger script at '/home/lin/.vs-debugger'.
11:34:39:958	
11:34:39:958	
11:34:39:959	初始化日志:
11:34:39:959	Determining user folder on remote system...
11:34:39:959	Checking for existing installation of debugging tools...
11:34:39:959	Downloading debugger launcher...
11:34:39:959	Creating debugger installation folder: /home/lin/.vs-debugger
11:34:39:959	Failed: 无法创建或访问指定的目录 /home/lin/.vs-debugger。
11:34:39:959	参数名: path
11:34:39:959	Unable to find debugger script at '/home/lin/.vs-debugger'.
11:34:39:959	Failed: Unable to find debugger script at '/home/lin/.vs-debugger'.
```

那就是 VisualStudio 创建 `.vs-debugger` 文件夹失败

或提示 Failed: The specified directory /home/lin/.vs-debugger could not be created or accessed. 等信息

可使用以下方法解决。回到 UOS 命令行里面，退出 sudo su 状态，使用用户权限创建 `~/.vs-debugger` 文件夹

```
exit

cd ~

mkdir .vs-debugger
```

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅17.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203535973-215050008.png)

以上的 exit 命令仅仅为了退出 sudo su 状态，如果你是新开的控制台，那请不要带上 exit 命令

完成以上命令之后，再次尝试在 VisualStudio 附加调试

预期能够看到正在启动调试适配器界面

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅18.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203536308-324036492.png)

看到此界面表示 VisualStudio 正在帮你部署调试环境，首次部署输入比较慢，大概需要在你的 UOS 下载 200MB 的内容

如果等太久，可以进入 `~/.vs-debugger` 输入 `du -sh` 命令查看大小，就可以知道是否正在下载

<!-- ![](image/UOS 开启 VisualStudio 远程调试 .NET 应用之旅/UOS 开启 VisualStudio 远程调试 .NET 应用之旅19.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203536660-274139608.png)

接下来就进入到了咱熟悉的 VisualStudio 调试 .NET 应用的状态了

试试点击 VisualStudio 的暂停按钮，看看线程和堆栈

此调试方式配合 dotnet-campus 开源的 SyncTool 工具使用更好，通过 SyncTool 将 Windows 上的 VisualStudio 构建输出内容同步到 Linux 设备上，然后使用远程调试方式进行调试

详细请参阅 [SyncTool 开源项目](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK)


## 参考博客

[VisualStudio 如何 SSH 远程调试 Linux 的 dotnet 应用的启动](https://blog.lindexi.com/post/VisualStudio-%E5%A6%82%E4%BD%95-SSH-%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95-Linux-%E7%9A%84-dotnet-%E5%BA%94%E7%94%A8%E7%9A%84%E5%90%AF%E5%8A%A8.html )
<!-- [VisualStudio 如何 SSH 远程调试 Linux 的 dotnet 应用的启动 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18238164 ) -->

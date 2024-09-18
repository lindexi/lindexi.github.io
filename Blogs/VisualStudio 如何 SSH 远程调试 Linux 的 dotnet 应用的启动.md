---
title: VisualStudio 如何 SSH 远程调试 Linux 的 dotnet 应用的启动
description: 本文将告诉大家，如何使用 VisualStudio 2022 工具，通过 SSH 远程调试运行在 Linux 系统上的 dotnet 系应用的启动过程
tags: VisualStudio,dotnet
category: 
---

<!-- CreateTime:2024/06/08 07:19:22 -->

<!-- 发布 -->
<!-- 博客 -->

本文写于 2024.06.07 如果你阅读本文的时间距离本文编写的时间过于长，那本文可能包含过期的知识

当前的 VisualStudio 2022 不能和在 Windows 一样，在 Linux 上一键进入构建且调试。只好通过远程调试的方式进行附加，而附加进程时，可能所需调试的是应用启动过程的逻辑，这将导致调试附加过去时，应用已经跑过了启动逻辑，导致无法进行调试

尽管编写一个 VisualStudio 插件可以很好的解决此问题，然而现在我没有那么有空，本文将告诉大家一个简单的方式，可以不依靠任何第三方工具，就使用现成的 VisualStudio 即可实现远程调试运行在 Linux 系统上的 dotnet 系应用的启动过程

开始之前，请参阅 [UOS 开启 VisualStudio 远程调试 .NET 应用之旅](https://blog.lindexi.com/post/UOS-%E5%BC%80%E5%90%AF-VisualStudio-%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95-.NET-%E5%BA%94%E7%94%A8%E4%B9%8B%E6%97%85.html ) 博客搭建好环境

在需要调试启动的 dotnet 应用里面，在 Program 的 Main 方法添加如下代码

```csharp
while (!Debugger.IsAttached)
{
    await Task.Delay(100);
}
Debugger.Break();
```

以上代码的含义就是，如果没有被附加，则不退出循环。被附加之后，进入 Debugger.Break 断点。如此即可让应用在没有附加调试时，一直卡住，不会执行启动逻辑，直到被附加调试之后，才进入断点，开发者就可以方便调试应用的启动

此方法适用于所有 dotnet 系应用，包括 MAUI 、Avalonia 、CPF、GtkWinform、UNO 等能在 Linux 上运行的 UI 框架的应用

例子的全部代码如下

```csharp
// Program.cs

using System.Diagnostics;

while (!Debugger.IsAttached)
{
    await Task.Delay(100);
}
Debugger.Break();

Console.WriteLine("Hello, World!");
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/fdc4816aba71f2cc6bfdcb6c7ca0bba2f0ac2b8c/X11/BalcawheldereFukenallbe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/fdc4816aba71f2cc6bfdcb6c7ca0bba2f0ac2b8c/X11/BalcawheldereFukenallbe) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin fdc4816aba71f2cc6bfdcb6c7ca0bba2f0ac2b8c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin fdc4816aba71f2cc6bfdcb6c7ca0bba2f0ac2b8c
```

获取代码之后，进入 X11/BalcawheldereFukenallbe 文件夹，即可获取到源代码

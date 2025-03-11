---
title: dotnet Cli sdk 什么是 Terminal Logger 以及如何禁用此功能
description: 在 dotnet 9 的 SDK 版本里面，将 Terminal Logger 功能设置为默认。这是一个令我烦恼的功能。我将在这篇博客里面和大家介绍这是一个什么功能，以及如何将此功能禁用
tags: dotnet
category: 
---

<!-- CreateTime:2025/03/11 07:07:22 -->

<!-- 发布 -->
<!-- 博客 -->

在 dotnet 9 默认开启的 Terminal Logger 功能，是在 dotnet 8 作为实验性引入的功能。核心原因是因为有开发者认为 dotnet sdk 在构建等过程中输出的内容太多了，干扰了程序猿的开发，期望能够自动折叠或删除不需要的输出内容。比如在 dotnet build 过程中，将在构建过程中刷刷刷显示日志的过程内容，如 Info 内容，但最终只包含构建过程，以及进行读秒。完成之后就只有警告和错误信息留下，其他信息全部删除

<!-- ![](image/dotnet Cli sdk 什么是 Terminal Logger 以及如何禁用此功能/dotnet Cli sdk 什么是 Terminal Logger 以及如何禁用此功能0.gif) -->
![](https://img2023.cnblogs.com/blog/1080237/202503/1080237-20250311070927413-763489157.gif)

```
PS C:\lindexi\Code\Foo> dotnet build
还原完成(0.2)
  Foo 已成功 (1.5) → bin\Debug\net9.0\Foo.dll

在 2.0 中生成 已成功
```

然而我日常开发很多开发任务都是和编译器相关的，这就意味着如果只剩下警告信息或错误信息，那对我调查问题是几乎完全没有帮助的

为了看到完全的日志，我就不断在 Windows 的 cmd 上，使用重定向输出方式，将输出内容放入到文件里面，如以下命令行示例，将构建输出信息重定向当前工作文件夹的 1.txt 文件里面。但这样的方式的效率很低，我需要不断读取本地文件才能知道构建日志信息

```
dotnet build > 1.txt
```

好在 dotnet 里面提供了对 Terminal Logger 功能的开关，可以使用 `--tl:off` 参数将其禁用，回到原本的完全日志输出形态

```
dotnet build --tl:off
```

执行以上之后的输出内容大概如下，可以看到还原过程等信息

```
PS C:\lindexi\Code\Foo> dotnet build --tl:off
  正在确定要还原的项目…
  已还原 C:\lindexi\Code\Foo\Foo.csproj (用时 36 毫秒)。
  Foo -> C:\lindexi\Code\Foo\bin\Debug\net9.0\Foo.dll

已成功生成。
    0 个警告
    0 个错误

已用时间 00:00:00.75
```

这项变更给许多库带来了坑点，原本很多库设计上都是简单控制台输出，现在为了更加方便定位问题，就需要直接打成 `warn:` 等开头的警告输出。如 DotNETBuildSDK 的 <https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/pull/167> 更改。但很多工具库都不能很好适应，因为日常输出过程中压根不知道后续会不会失败，如果都是 Info 等级输出，等到构建失败的时候，却会让开发者缺乏信息。特别是对新手开发者不友好，新手开发者一截图，都会发现毫无有用信息，有用的信息都被省略掉了

记录本文的目的是因为太难搜到了，我压根不知道这个功能叫什么。即使我在 [Breaking change: Terminal logger is default - .NET - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/9.0/terminal-logger ) 这篇文档经过了非常多次，我都无法联系起来

为了方便大家找到本文，我添加了一些关键词

禁用 dotnet 折叠构建日志

禁止 dotnet sdk 删除构建日志

dotnet 命令行输出构建步骤时间

dotnet 命令行步骤时间

dotnet 命令行构建读秒

dotnet 控制台只输出重要信息

dotnet 控制台省略日志

dotnet 控制台省略输出

dotnet 控制台行折叠

dotnet build 只输出重要信息

dotnet build 输出 Info 等级日志信息

dotnet 控制台删除信息等级日志输出

dotnet 命令行 -v:diag 内容看不见

dotnet 只输出警告

dotnet build 只输出警告

dotnet 只输出错误

dotnet build 只输出错误

参考文档：

- [Enable the new TerminalLogger in .NET 8 SDK automatically - Meziantou's blog](https://www.meziantou.net/enable-the-new-terminallogger-in-dotnet-8-sdk-automatically.htm )

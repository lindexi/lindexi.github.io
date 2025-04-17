---
title: Avalonia 11.1 获取平台调用的窗口的方法
description: 本文和大家介绍如何在 11.1 版本的 Avalonia 里获取平台调用的窗口的方法，如 Windows 获取窗口句柄，在 Linux 下获取 X11 的 xid 窗口信息
tags: Avalonia
category: 
---

<!-- CreateTime:2024/08/09 09:07:49 -->

<!-- 发布 -->
<!-- 博客 -->

在拿到任意的 Avalonia 的 Visual 元素，可通过 TopLevel 的 GetTopLevel 方法获取到其窗口。由于 Avalonia 是一个跨平台的 UI 框架，因此不能假定一定存在窗口，于是这里的 GetTopLevel 只是返回一个名为 TopLevel 的对象

在桌面平台里，这里的 TopLevel 对象就是窗口对象

获取到 TopLevel 对象之后，即可再调用 TryGetPlatformHandle 方法，尝试获取平台信息，代码如下

```csharp
        var topLevel = TopLevel.GetTopLevel(this)!;

        // 通过窗口获取，方法更加简单：
        var handle = topLevel.TryGetPlatformHandle()!;
        Console.WriteLine($"X11 xid {handle.Handle}");
```

除了以上方法之外，还可以反射 PlatformImpl 获取。其代码实现如下

```csharp
        var platformImpl = topLevel.PlatformImpl;

        var type = platformImpl.GetType();

        var propertyInfo = type.GetProperty("Handle", BindingFlags.Instance | BindingFlags.Public);

        var value = propertyInfo.GetValue(platformImpl);

        Debug.Assert(value is IPlatformHandle);

        if (value is PlatformHandle platformHandle)
        {
            var x11Handler = platformHandle.Handle;
            Console.WriteLine(x11Handler);
        }
        else if(value is IPlatformHandle platformHandle2)
        {
            // 当前在 Windows 的没有明确的类型，是一个放在 WindowImpl 类中的 WindowImplPlatformHandle 内部类
            var hwnd = platformHandle2.Handle;
            Console.WriteLine(hwnd);
        }
```

通过以上代码可以看到 Avalonia 实现的不一致，在 Windows 下的实现里面，没有明确的 PlatformHandle 类型

好在后续版本已经对此进行变更，允许从 PlatformImpl 属性直接获取 Handle 属性，如此设计将会更加预期。也就是意味着在 11.1 之后的版本，预期可以不用写反射，即可直接从 PlatformImpl 属性获取到 Handle 属性

以上代码使用反射获取到的 Handle 和 TryGetPlatformHandle 获取到的是相同的

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/5b958b3dc584ebf34351c4c0015bce17bb5b05d4/AvaloniaIDemo/NaiqojunefeakeeLurkarlabefije) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5b958b3dc584ebf34351c4c0015bce17bb5b05d4/AvaloniaIDemo/NaiqojunefeakeeLurkarlabefije) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5b958b3dc584ebf34351c4c0015bce17bb5b05d4
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5b958b3dc584ebf34351c4c0015bce17bb5b05d4
```

获取代码之后，进入 AvaloniaIDemo/NaiqojunefeakeeLurkarlabefije 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

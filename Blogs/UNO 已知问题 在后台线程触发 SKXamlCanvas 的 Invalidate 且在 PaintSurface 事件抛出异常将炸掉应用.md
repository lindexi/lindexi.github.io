---
title: UNO 已知问题 在后台线程触发 SKXamlCanvas 的 Invalidate 且在 PaintSurface 事件抛出异常将炸掉应用
description: 本文记录一个 UNO 已知问题，在 UNO 里面可以利用 SKXamlCanvas 对接 Skia 绘制到应用里面。如果此时在后台线程里面调用 SKXamlCanvas 的 Invalidate 触发界面的重新刷新，但在具体的执行绘制 PaintSurface 事件里面对外抛出异常，将会导致应用炸掉
tags: UNO
category: 
---

<!-- CreateTime:2024/1/19 15:20:33 -->

<!-- 博客 -->
<!-- 发布 -->

背景： 我准备在 UNO 里面将 Microsoft.Maui.Graphics 对接进入，于是就用到了 SKXamlCanvas 控件。详细请看 <https://github.com/unoplatform/uno/discussions/15097>

当前行为： 当我使用 SKXamlCanvas 时，如果我在 PaintSurface 事件里面抛出任何异常，且当前的 PaintSurface 事件是由后台线程触发的，那将导致我的进程崩溃

预期行为：即使在 PaintSurface 事件里面抛出任何异常，应用程序也可以正常工作且收集到异常，比如通过 TaskScheduler.UnobservedTaskException 事件收集到异常

复现步骤：

1. 添加 SKXamlCanvas 到 xaml 里
2. 订阅 SKXamlCanvas 的 PaintSurface 事件，且在事件实现方法抛出异常
3. 在后台线程调用 SKXamlCanvas 的 Invalidate 方法

核心的代码实现如下

在 XAML 添加 SKXamlCanvas 控件

```xml
  xmlns:sk="using:SkiaSharp.Views.Windows"

  <sk:SKXamlCanvas x:Name="Canvas" PaintSurface="OnPaintSurface" />
```

在后台代码里面使用后台线程调用 SKXamlCanvas 的 Invalidate 方法，且在 OnPaintSurface 抛出异常

```csharp
            Task.Run(() =>
            {
                Canvas.Invalidate();
            });

        private void OnPaintSurface(object sender, SKPaintSurfaceEventArgs e)
        {
            throw new Exception();
        }
```

本文以上的复现代码放在[github](https://github.com/lindexi/lindexi_gd/tree/dde76effc23ebb9ee974b6ec276b242c39a50bdf/JagobawearjiNeewhiqakerki) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/dde76effc23ebb9ee974b6ec276b242c39a50bdf/JagobawearjiNeewhiqakerki) 欢迎访问

可以通过如下方式获取以上的复现代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin dde76effc23ebb9ee974b6ec276b242c39a50bdf
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin dde76effc23ebb9ee974b6ec276b242c39a50bdf
```

获取代码之后，进入 JagobawearjiNeewhiqakerki 文件夹

报告地址： <https://github.com/unoplatform/uno/issues/15123>

原因：

这是由于在 [SkiaSharp](https://github.com/mono/SkiaSharp) 里面的错误实现导致踩到 dotnet 的另一个已知问题导致的。在 SKXamlCanvas 的具体实现里面，通过 async void 等待执行结果，而根据 dotnet 的已知问题可以知道，在 async void 收到任何异常都会导致进程崩溃，此行为详细请参阅 [dotnet 警惕 async void 线程顶层异常](https://blog.lindexi.com/post/dotnet-%E8%AD%A6%E6%83%95-async-void-%E7%BA%BF%E7%A8%8B%E9%A1%B6%E5%B1%82%E5%BC%82%E5%B8%B8.html )

解决方法：

此问题已经被我修复，详细请看 [Avoid async void in SKXamlCanvas. by lindexi · Pull Request #2720 · mono/SkiaSharp](https://github.com/mono/SkiaSharp/pull/2720 )

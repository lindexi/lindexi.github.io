---
title: Avalonia 在 X11 下使用软渲染的方法
description: 本文的方法适用于 11.0 的 Avalonia 版本

<!--more-->

tags: 
category: 
---

<!-- CreateTime:2024/07/23 07:27:20 -->

<!-- 发布 -->
<!-- 博客 -->

只需在 Program.cs 的 BuildAvaloniaApp 方法里面配置 X11PlatformOptions 即可，代码如下

```csharp
            .With(new X11PlatformOptions()
            {
                RenderingMode = new List<X11RenderingMode>()
                {
                    X11RenderingMode.Software
                }
            }
```

修改之后的 Program.cs 大概代码如下

```csharp
using System;
using System.Collections.Generic;

using Avalonia;
using Avalonia.Media;

namespace NelkiwalweawhabairJewugekayfaylugere.Desktop;

class Program
{
    // Initialization code. Don't use any Avalonia, third-party APIs or any
    // SynchronizationContext-reliant code before AppMain is called: things aren't initialized
    // yet and stuff might break.
    [STAThread]
    public static void Main(string[] args) => BuildAvaloniaApp()
        .StartWithClassicDesktopLifetime(args);

    // Avalonia configuration, don't remove; also used by visual designer.
    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>()
            .UsePlatformDetect()
            .WithInterFont()
            .LogToTrace()
            // 修复麒麟丢失字体
            .With(new FontManagerOptions()
            {
                DefaultFamilyName = "Noto Sans CJK SC",
                FontFallbacks =
                [
                    new FontFallback { FontFamily = "文泉驿正黑" },
                    new FontFallback { FontFamily = "DejaVu Sans" },
                ],
            })
            .With(new X11PlatformOptions()
            {
                RenderingMode = new List<X11RenderingMode>()
                {
                    X11RenderingMode.Software
                }
            });

}
```

以上的代码设置字体是为了解决在麒麟系统无法显示中文的问题或运行进程失败的问题。详细请看 [dotnet 解决 UNO 在 OpenKylin 麒麟系统运行找不到默认字体启动失败](https://blog.lindexi.com/post/dotnet-%E8%A7%A3%E5%86%B3-UNO-%E5%9C%A8-OpenKylin-%E9%BA%92%E9%BA%9F%E7%B3%BB%E7%BB%9F%E8%BF%90%E8%A1%8C%E6%89%BE%E4%B8%8D%E5%88%B0%E9%BB%98%E8%AE%A4%E5%AD%97%E4%BD%93%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5.html )
<!-- [dotnet 解决 UNO 在 OpenKylin 麒麟系统运行找不到默认字体启动失败 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18268131 ) -->

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8402eaca48804ec2418b459f540e5e48d1109a23/AvaloniaIDemo/NelkiwalweawhabairJewugekayfaylugere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8402eaca48804ec2418b459f540e5e48d1109a23/AvaloniaIDemo/NelkiwalweawhabairJewugekayfaylugere) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8402eaca48804ec2418b459f540e5e48d1109a23
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8402eaca48804ec2418b459f540e5e48d1109a23
```

获取代码之后，进入 AvaloniaIDemo/NelkiwalweawhabairJewugekayfaylugere 文件夹，即可获取到源代码

更多 Avalonia 以及 X11 等相关技术，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

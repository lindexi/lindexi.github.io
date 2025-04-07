---
title: dotnet C# 判断应用程序在 Wine 里运行的方法
description: 本文将记录如何在 C# dotnet 代码里面判断当前进程运行在 Wine 里面
tags: dotnet,C#
category: 
---

<!-- CreateTime:2025/04/04 10:05:02 -->

<!-- 发布 -->
<!-- 博客 -->

本文记录的方法由 [lsj](https://blog.sdlsj.net) 提供，我只是代为记录的工具人

根据 Wine 官方文档： <https://gitlab.winehq.org/wine/wine/-/wikis/Developer-FAQ#how-can-i-detect-wine>

可通过 ntdll 导出的 wine_get_version 方法进行判断。正常不运行在 Wine 里面的程序，拿到的 ntdll 是不会存在 wine_get_version 方法导出的。换句话说就是能调用到 wine_get_version 方法则证明跑在 Wine 环境里面

以下是我新建的 .NET 9 控制台项目代码，理论上以下代码在低至 .NET Core 3.0 里面也能跑，但我自己没试过哈

```csharp
using System.Diagnostics;
using System.Runtime.InteropServices;

if (WineDetector.IsRunningOnWine())
{
    Console.WriteLine($"在 Wine 里运行");
}
else
{
    Console.WriteLine($"不在 Wine 里运行");
}

class WineDetector
{
    public static bool IsRunningOnWine()
    {
        // https://gitlab.winehq.org/wine/wine/-/wikis/Developer-FAQ#how-can-i-detect-wine
        // If you still really want to detect Wine, check whether ntdll exports the function wine_get_version.
        try
        {
            var handle = NativeLibrary.Load("ntdll.dll");

            if (NativeLibrary.TryGetExport(handle, "wine_get_version", out var address))
            {
                Debug.Assert(address != IntPtr.Zero);
                return true;
            }
            else
            {
                return false;
            }
        }
        catch
        {
            // 不应该发生，Windows 下 ntdll.dll 不可能不存在
            return false;
        }
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/358c342b5c92600c96b1d974420a2212453254b6/Workbench/HaherefeanernawafalCochereliwemkee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/358c342b5c92600c96b1d974420a2212453254b6/Workbench/HaherefeanernawafalCochereliwemkee) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 358c342b5c92600c96b1d974420a2212453254b6
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 358c342b5c92600c96b1d974420a2212453254b6
```

获取代码之后，进入 Workbench/HaherefeanernawafalCochereliwemkee 文件夹，即可获取到源代码

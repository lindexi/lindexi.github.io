---
title: Avalonia 制作 AOT 单文件
description: 对 Avalonia 进行 AOT 发布时，会发现存在几个库文件无法被打进入到 exe 可执行文件里面，于是进行分发的时候又需要进行压缩。现在很多用户已经不懂如何解压缩了，我就在想着如何只发布单个文件
tags: Avalonia
category: 
---

<!-- 发布 -->
<!-- 博客 -->

本文提供的方法在 11.3.2 的 Avalonia 版本实验成功，支持 Windows x86 应用程序 AOT 发布为完全单文件。预期对 windows x64 也能成功 AOT 发布完全单文件

实现方式如下

先正常对应用项目进行 AOT 发布，发布之后，预期会多出以下几个 DLL 文件：

- av_libglesv2.dll
- libHarfBuzzSharp.dll
- libSkiaSharp.dll

将这几个 DLL 文件拷贝出来，随便找个文件夹放。如我这里就放在 `C:\lindexi\Library\Avalonia_win-x86\` 文件夹里

回到项目里面，修改 csproj 项目文件，添加对存放的 DLL 文件的引用，设置为嵌入程序集资源，大概代码如下

```xml
  <ItemGroup>
    <EmbeddedResource Include="C:\lindexi\Library\Avalonia_win-x86\*.dll" LinkBase="Assets\win-x86" />
  </ItemGroup>
```

如果不知道这部分代码怎么写，可以在本文末尾获取本文所有代码的下载方法，拉取我的代码了解具体的代码

接着修改 Program.cs 的 Main 函数，将原本写在 Main 函数里面的 Avalonia 调用抽一个独立的方法。如我这里放在 RunAvalonia 方法里面。独立方法的作用是防止在进入 Main 时立刻碰到 Avalonia 类型，导致类型快速初始化，类型快速初始化时可能会碰到某些基础库引用，此时基础库还没被释放出来，就可能导致异常

再编写一个名为 LoadNativeLib 的方法，方法代码如下

```csharp
    private static void LoadNativeLib()
    {
        var assembly = typeof(Program).Assembly;
        var manifestResourceNames = assembly.GetManifestResourceNames();

        var platform = "win_x86";
        var platformResource = $"CibairfejeballChecekayral.Desktop.Assets.{platform}.";
        var folder = Directory.CreateDirectory(Path.Join(AppContext.BaseDirectory, platform));

        foreach (var manifestResourceName in manifestResourceNames)
        {
            if (manifestResourceName.StartsWith(platformResource))
            {
                using var manifestResourceStream = assembly.GetManifestResourceStream(manifestResourceName)!;
                var fileName = manifestResourceName[platformResource.Length..];
                var file = Path.Join(folder.FullName, fileName);
                if (!File.Exists(file))
                {
                    using var fileStream = File.OpenWrite(file);
                    manifestResourceStream.CopyTo(fileStream);
                }

                NativeLibrary.Load(file);
            }
        }
    }
```

通过以上代码可以看到核心实现就是将嵌入程序集里面的几个 DLL 释放出来，我这里只处理了 x86 的情况，其他情况还请大家自行判断处理。从程序集里面取出 DLL 写入到文件之后，调用 NativeLibrary.Load 方法即可执行加载

完成之后，再次进行 AOT 发布，此时只取发布出来的 exe 一个文件放在另一个空白文件夹里面双击运行，可见此时可以成功正常运行

通过如此方式即可实现在 Avalonia 里面进行独立 AOT 成单个文件，非常方便小工具的分发

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/21d0bbfbb70ff2274d74357b90c80ea32656c727/AvaloniaIDemo/CibairfejeballChecekayral) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/21d0bbfbb70ff2274d74357b90c80ea32656c727/AvaloniaIDemo/CibairfejeballChecekayral) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 21d0bbfbb70ff2274d74357b90c80ea32656c727
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 21d0bbfbb70ff2274d74357b90c80ea32656c727
```

获取代码之后，进入 AvaloniaIDemo/CibairfejeballChecekayral 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

# Avalonia 解决渲染高清大图帧率下降问题

本文将和大家介绍通过加大设置最大 GPU 资源缓存的方式解决渲染高清大图帧率下降问题

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

这个问题开始是我在 GitHub 上询问的，详细请参阅： [Low Rendering Frame Rate with High-Resolution Images in Avalonia Image Control · Issue #20658 · AvaloniaUI/Avalonia](https://github.com/AvaloniaUI/Avalonia/issues/20658 )

在 Avalonia 里面，打开高清大图的时候，将会发现随着图片的分辨率越高，画面渲染帧率越低

为此我还编写了一个测试程序，可以自动生成 2K 图和 4K 图等不同分辨率的图片，尝试加载不同的图片可以看到渲染帧率的变化

通过加大设置最大 GPU 资源缓存的方式可以解决此问题，加大的方式是在 Program.cs 里面的 BuildAvaloniaApp 方法里添加如下代码

```csharp
            .With(new SkiaOptions()
            {
                MaxGpuResourceSizeBytes = 一个大一些的数
            })
```

更改之后的代码大概如下


```csharp
    // Avalonia configuration, don't remove; also used by visual designer.
    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>()
            .With(new SkiaOptions()
            {
                MaxGpuResourceSizeBytes = long.MaxValue / 2 （不一定是越大越好，小心爆）
            })
            .UsePlatformDetect()
            .WithInterFont()
            .LogToTrace();
```

如果不知道以上代码具体改在哪里，可以按照以下方式拉取本文的用到的代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1b0c0dc806791090d079b089ec0e2078fec8d4fb/AvaloniaIDemo/YewhuhuciDacayfudawi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1b0c0dc806791090d079b089ec0e2078fec8d4fb/AvaloniaIDemo/YewhuhuciDacayfudawi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1b0c0dc806791090d079b089ec0e2078fec8d4fb
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1b0c0dc806791090d079b089ec0e2078fec8d4fb
```

获取代码之后，进入 AvaloniaIDemo/YewhuhuciDacayfudawi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


# dotnet C# 使用 Vortice 创建 Direct2D1 的 ID2D1SolidColorBrush 纯色画刷

在进行 D2D 绘制文本或者是形状的时候，期望填充某个颜色，就需要用到 ID2D1SolidColorBrush 纯色画刷，在绘制的时候通过纯色画刷进行填充颜色。本文将告诉大家如何使用 Vortice 库创建 ID2D1SolidColorBrush 纯色画刷

<!--more-->



<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 发布 -->

本文属于使用 Vortice 调用 DirectX 系列博客，也属于 DirectX 系列博客，本文属于入门级博客，但在阅读本文之前，期望大家了解了 DirectX 的基础概念。本文的前置博客是 [dotnet C# 使用 Vortice 支持 Direct2D1 离屏渲染](https://blog.lindexi.com/post/dotnet-C-%E4%BD%BF%E7%94%A8-Vortice-%E6%94%AF%E6%8C%81-Direct2D1-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html )

为了演示方便，本文是采用 Direct2D1 离屏渲染的方式，将结果输出到本地图片文件。如何进行离屏渲染请看 [dotnet C# 使用 Vortice 支持 Direct2D1 离屏渲染](https://blog.lindexi.com/post/dotnet-C-%E4%BD%BF%E7%94%A8-Vortice-%E6%94%AF%E6%8C%81-Direct2D1-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html )


在获取到 ID2D1RenderTarget 之后，可以通过 ID2D1RenderTarget 的 CreateSolidColorBrush 方法进行创建纯色画刷。创建时需要传入颜色结构体

先创建颜色结构体，这里使用随机创建

```csharp
var color = new Color4(GetRandom(), GetRandom(), GetRandom());

byte GetRandom() => (byte) Random.Shared.Next(255);
```

接着调用 CreateSolidColorBrush 方法创建 ID2D1SolidColorBrush 纯色画刷

```csharp
        using D2D.ID2D1SolidColorBrush brush = renderTarget.CreateSolidColorBrush(color);
```

如此即可获取到纯色画刷

可以用此纯色画刷进行填充绘制的内容，例如绘制圆形填充颜色

```csharp
        var width = 1000;
        var height = 1000;

            var radiusX = 5;
            var radiusY = 5;
            renderTarget.DrawEllipse(new D2D.Ellipse(new Vector2(Random.Shared.Next(width - radiusX), Random.Shared.Next(height - radiusY)), radiusX, radiusY), brush, 2);
```

全部代码如下

```csharp
using System.Diagnostics;
using System.Numerics;

using Vortice.Mathematics;
using Vortice.WIC;

using D2D = Vortice.Direct2D1;
using PixelFormat = Vortice.DCommon.PixelFormat;

namespace WakolerwhaKanicabirem;

class Program
{
    // 设置可以支持 Win7 和以上版本。如果用到 WinRT 可以设置为支持 win10 和以上。这个特性只是给 VS 看的，没有实际影响运行的逻辑
    static void Main(string[] args)
    {
        // 对接 D2D 需要创建工厂
        using D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();

        var renderTargetProperties = new D2D.RenderTargetProperties(PixelFormat.Premultiplied);

        using var wicImagingFactory = new IWICImagingFactory();
        var width = 1000;
        var height = 1000;
        using var wicBitmap =
            wicImagingFactory.CreateBitmap(width, height, Win32.Graphics.Imaging.Apis.GUID_WICPixelFormat32bppPBGRA);

        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory.CreateWicBitmapRenderTarget(wicBitmap, renderTargetProperties);

        using var renderTarget = d2D1RenderTarget;
        var stopwatch = Stopwatch.StartNew();
        // 开始绘制逻辑
        renderTarget.BeginDraw();

        // 随意创建颜色
        var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255),
            (byte) Random.Shared.Next(255));
        renderTarget.Clear(color);
        color = new Color4(GetRandom(), GetRandom(), GetRandom());
        using D2D.ID2D1SolidColorBrush brush = renderTarget.CreateSolidColorBrush(color);

        for (int i = 0; i < 10; i++)
        {
            var radiusX = 5;
            var radiusY = 5;
            renderTarget.DrawEllipse(new D2D.Ellipse(new Vector2(Random.Shared.Next(width - radiusX), Random.Shared.Next(height - radiusY)), radiusX, radiusY), brush, 2);
        }

        stopwatch.Stop();
        Console.WriteLine($"Draw: {stopwatch.ElapsedMilliseconds}");
        stopwatch.Restart();

        renderTarget.EndDraw();

        stopwatch.Stop();
        Console.WriteLine($"EndDraw: {stopwatch.ElapsedMilliseconds}");
        stopwatch.Restart();

        byte GetRandom() => (byte) Random.Shared.Next(255);

        var file = @"D2D.png";
        using (var fileStream = File.OpenWrite(file))
        {
            using var wicBitmapEncoder =
                wicImagingFactory.CreateEncoder(Win32.Graphics.Imaging.Apis.GUID_ContainerFormatPng);

            wicBitmapEncoder.Initialize(fileStream);
            using var wicFrameEncode = wicBitmapEncoder.CreateNewFrame(out var _);
            wicFrameEncode.Initialize();
            wicFrameEncode.WriteSource(wicBitmap);
            wicFrameEncode.Commit();
            wicBitmapEncoder.Commit();
        }
    }
}
```

执行以上代码的绘制结果如下

<!-- ![](image/dotnet C# 使用 Vortice 创建 Direct2D1 的 ID2D1SolidColorBrush 纯色画刷/dotnet C# 使用 Vortice 创建 Direct2D1 的 ID2D1SolidColorBrush 纯色画刷0.png) -->

![](http://image.acmx.xyz/lindexi%2FD2D.png)

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/471614ba9a8981c3e23041804785ff77f23dac82/WakolerwhaKanicabirem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/471614ba9a8981c3e23041804785ff77f23dac82/WakolerwhaKanicabirem) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 471614ba9a8981c3e23041804785ff77f23dac82
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 471614ba9a8981c3e23041804785ff77f23dac82
```

获取代码之后，进入 WakolerwhaKanicabirem 文件夹

渲染部分，关于 SharpDx 使用，包括入门级教程，请参阅：

- [WPF 使用 SharpDx 渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
- [SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

在 WPF 框架的渲染部分，请参阅： [WPF 底层渲染_lindexi_gd的博客-CSDN博客](https://blog.csdn.net/lindexi_gd/category_9276313.html?spm=1001.2014.3001.5482 )

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

交流 Vortice 技术，欢迎加群： 622808968




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
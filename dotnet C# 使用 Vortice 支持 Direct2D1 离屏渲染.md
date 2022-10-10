# dotnet C# 使用 Vortice 支持 Direct2D1 离屏渲染

本文告诉大家如何使用 Vortice 进行 D2D 的离屏渲染功能，本文将在一个纯控制台无窗口的应用下，使用 Direct2D1 进行离屏绘制，将绘制结果保存为本地图片文件

<!--more-->
<!-- 博客 -->
<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 发布 -->

本文属于使用 Vortice 调用 DirectX 系列博客，也属于 DirectX 系列博客，本文属于入门级博客，但在阅读本文之前，期望大家了解了 DirectX 的基础概念

本文使用的 Vortice 是 SharpDx 的代替品，是对 DirectX 的底层 C# 封装。使用 Vortice 底层库，能让 C# 代码比较方便的和 DirectX 对接。尽管本文使用的是 Vortice 库来调用 DirectX 相关的接口，但不代表着只有 Vortice 库能做此实现，可以将 Vortice 换成其他的对 DirectX 封装的库，例如 SharpDx 或者是 Silk.NET 等，更换之后只是调用的方法或者是参数等稍微有点不相同，但是实现思路都是相同的

使用 Direct2D1 离屏渲染技术，可以进行脱离具体的窗口调用渲染，可以不需要占用主线程的时间，采用后台线程驱动执行渲染。可以实现在某些性能敏感的业务上，预先准备好渲染内容，从而提升性能等

新建一个 dotnet 6 的控制台项目，咱将在此新建的项目里面完成一个简单的 Direct2D1 离屏渲染的控制台应用。本文不会贴所有的代码，如果按照本文给出的代码构建不通过或者遇到其他问题，还请到本文末尾获取所有源代码，将源代码拉到本地构建

按照惯例，在开始之前，先通过 NuGet 安装必要的库。在 dotnet 6 项目里，采用 SDK 风格的 csproj 项目文件格式，可以通过编辑项目文件的方式快速安装 NuGet 库。可以在 VisualStudio 里，右击项目，点击编辑项目文件，或者双击项目都可以，替换项目文件为以下代码即可完成安装

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Vortice.Direct2D1" Version="2.1.32" />
    <PackageReference Include="Vortice.Win32" Version="1.6.2" />
  </ItemGroup>
</Project>
```

以上代码安装了 Vortice.Direct2D1 库用来对接 Direct2D1 的逻辑，安装了 Vortice.Win32 用来辅助处理杂项逻辑

打开 Program.cs 文件，开始编写离线渲染逻辑。开始编写代码之前，先引用命名空间

```csharp
using Vortice.Mathematics;
using Vortice.WIC;
using D2D = Vortice.Direct2D1;
using PixelFormat = Vortice.DCommon.PixelFormat;
```

大家都知道，使用 D2D 时，最重要的就是获取到 ID2D1RenderTarget 用来作为绘制的画布。创建 ID2D1RenderTarget 的方法有很多个，本文是通过 ID2D1Factory1 工厂调用 CreateWicBitmapRenderTarget 方法，从一个 IWICBitmap 创建的

也就是说想要获取到 ID2D1RenderTarget 进行绘制，就需要能先拿到 IWICBitmap 类型的对象。创建 IWICBitmap 类型的对象需要通过 WIC 工厂进行创建。于是先创建工厂

```csharp
        using var wicImagingFactory = new IWICImagingFactory();
```

通过 IWICImagingFactory 工厂创建 IWICBitmap 对象，需要调用 CreateBitmap 方法，传入尺寸和颜色格式。颜色格式里面只有一些是 D2D 支持的，本文这里采用常用的 PixelFormat32bppPBGRA 格式

```csharp
        using IWICBitmap wicBitmap =
            wicImagingFactory.CreateBitmap(1000, 1000, Win32.Graphics.Imaging.Apis.GUID_WICPixelFormat32bppPBGRA);
```

获取到 IWICBitmap 类型的对象，即可开始通过 D2D 工厂创建 ID2D1RenderTarget 画布。先创建 D2D 工厂

```csharp
        using D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();
```

接着设置渲染参数

```csharp
        var renderTargetProperties = new D2D.RenderTargetProperties(PixelFormat.Premultiplied);
```

创建 ID2D1RenderTarget 对象

```csharp
        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory.CreateWicBitmapRenderTarget(wicBitmap, renderTargetProperties);
```

以上就是最核心的步骤了，获取到 ID2D1RenderTarget 对象，即可开始 D2D 的绘制逻辑，如下面代码，修改画布颜色

```csharp
        using var renderTarget = d2D1RenderTarget;
        // 开始绘制逻辑
        renderTarget.BeginDraw();

        // 随意创建颜色
        var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255),
            (byte) Random.Shared.Next(255));
        renderTarget.Clear(color);

        renderTarget.EndDraw();
```

如此即可将内容绘制到 IWICBitmap 上

接下来是将 IWICBitmap 的内容保存到本地的图片，保存 IWICBitmap 需要先对 IWICBitmap 进行编码，编码时需要使用 WIC 工厂创建编码器，接着传入编码的格式和编码的输出

先打开一个文件用来存放编码的输出

```csharp
        var file = @"D2D.png";
        using (var fileStream = File.OpenWrite(file))
        {
            // 忽略代码
        }
```

通过 WIC 工厂创建编码器，设置编码的格式是 png 格式

```csharp
            using var wicBitmapEncoder =
                wicImagingFactory.CreateEncoder(Win32.Graphics.Imaging.Apis.GUID_ContainerFormatPng);
```

设置编码的输出到文件

```csharp
            wicBitmapEncoder.Initialize(fileStream);
```

从编码器创建出一张图片

```csharp
            using var wicFrameEncode = wicBitmapEncoder.CreateNewFrame(out var _);
            wicFrameEncode.Initialize();
```

将上文完成绘制的 IWICBitmap 输入到编码器里面

```csharp
            wicFrameEncode.WriteSource(wicBitmap);
```

完成逻辑之后提交一下

```csharp
            wicFrameEncode.Commit();
            wicBitmapEncoder.Commit();
```

如此执行完成，即可将绘制的内容保存到本地文件里。这就是本文的采用 D2D 进行离屏绘制的方法

想不开的话，可以测试一下调用渲染时是否能跑满 GPU 资源，稍微更改一下渲染的代码，从原本的调用 Clear 修改颜色，修改为以下逻辑

```csharp
        using var renderTarget = d2D1RenderTarget;
        var stopwatch = Stopwatch.StartNew();
        while (true)
        {
            // 开始绘制逻辑
            renderTarget.BeginDraw();

            // 随意创建颜色
            var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255),
                (byte) Random.Shared.Next(255));
            renderTarget.Clear(color);
            color = new Color4(GetRandom(), GetRandom(), GetRandom());
            using var brush = renderTarget.CreateSolidColorBrush(color);

            for (int i = 0; i < 1000; i++)
            {
                renderTarget.DrawEllipse(new D2D.Ellipse(new Vector2(GetRandom(), GetRandom()), 5, 5), brush, 2);
            }

            stopwatch.Stop();
            Console.WriteLine($"Draw: {stopwatch.ElapsedMilliseconds}");
            stopwatch.Restart();

            renderTarget.EndDraw();

            stopwatch.Stop();
            Console.WriteLine($"EndDraw: {stopwatch.ElapsedMilliseconds}");
            stopwatch.Restart();

            byte GetRandom() => (byte) Random.Shared.Next(255);
        }
```

尝试运行代码，看看任务管理器里面，显示当前进程是否有用到 GPU 资源，以及占用了多少 GPU 资源

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/bb1f1f3db2cf7317341e830d1e3adb14df67a71e/WakolerwhaKanicabirem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bb1f1f3db2cf7317341e830d1e3adb14df67a71e/WakolerwhaKanicabirem) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bb1f1f3db2cf7317341e830d1e3adb14df67a71e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bb1f1f3db2cf7317341e830d1e3adb14df67a71e
```

获取代码之后，进入 WakolerwhaKanicabirem 文件夹

渲染部分，关于 SharpDx 使用，包括入门级教程，请参阅：

- [WPF 使用 SharpDx 渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
- [SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

在 WPF 框架的渲染部分，请参阅： [WPF 底层渲染_lindexi_gd的博客-CSDN博客](https://blog.csdn.net/lindexi_gd/category_9276313.html?spm=1001.2014.3001.5482 )

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

交流 Vortice 技术，欢迎加群： 622808968

---
title: WPF 基础绘图 创建和加工图片
description: 本文将从控制台开始，告诉大家一个非常简单的 WPF 基础绘图方法，通过本文的方法可以调用 WPF 上层人类友好的方法，充分利用 GPU 资源，创建或加工图片，最终结果可以输出到本地文件，可支持编码出多种不同的图片格式
tags: WPF
category: 
---

<!-- CreateTime:2024/04/14 12:07:33 -->

<!-- 发布 -->
<!-- 博客 -->

本文仅用到 WPF 的多媒体渲染层，在 WPF 的这一层上的 API 是人类友好的，直接咱使用的是就是熟悉的 DrawingContext 类型。通过 DrawingContext 进行画线、画矩形、画几何、画图片、画文字，进行裁剪、变换、加特效等等，即可绘制出绚酷的界面效果。经由 RenderTargetBitmap 和 BitmapEncoder 即可将界面编码为图片输出到文件里面

从本质上讲，本文的方法里面只是将 WPF 当成一个对 WIC (Windows Imaging Component) 层以及 DirectX 层的高级封装库，没有用到 WPF 更多好用的功能

回到主题，先创建一个控制台项目，在控制台项目里面打上 WPF 的负载，加上之后的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UseWpf>true</UseWpf>
  </PropertyGroup>

</Project>
```

在控制台里面要么在 Main 函数标记 STAThread 特性，要么自己新建一个 STA 线程，由于我之前的博客大量介绍的都是在 Main 函数上标记 STAThread 特性，于是我准备在这篇博客换一个玩法，就是自己新建一个 STA 线程。相对来说新建一个线程的代码会稍微多一点点

```csharp
var thread = new Thread(() =>
{
    ... // 等待添加更多代码
});

thread.SetApartmentState(ApartmentState.STA);
thread.Start();
```

完成线程创建之后，即可在以上的 `等待添加更多代码` 里面加上 WPF 应用的初始化，在 WPF 应用的初始化里面自动将包含大量的初始化工作，简单的代码如下

```csharp
var thread = new Thread(() =>
{
    var application = new Application();
    application.Startup += Application_Startup;
    application.Run();
});
```

在 Startup 事件里面执行真正的处理绘图相关逻辑，以上代码的 Application_Startup 的基础实现逻辑如下

先创建 DrawingVisual 对象，通过此对象即可获取到 DrawingContext 以执行绘图逻辑

```csharp
void Application_Startup(object sender, StartupEventArgs e)
{
    var drawingVisual = new DrawingVisual();
    ... // 等待添加更多代码
}
```

从 DrawingVisual 里面获取 DrawingContext 的代码如下

```csharp
    var drawingVisual = new DrawingVisual();
    using (DrawingContext drawingContext = drawingVisual.RenderOpen())
    {
        ... // 等待添加更多代码
    }
```

通过 DrawingContext 的 DrawXxx 系列方法即可绘制出有趣的界面效果，比如本文例子中绘制简单的矩形的代码

```csharp
    using (var drawingContext = drawingVisual.RenderOpen())
    {
        drawingContext.DrawRectangle(Brushes.Black, pen: null, new Rect(0, 0, 1024, 768));
    }
```

实际使用中，可以在此替换为你所需的各种代码，包括先绘制图片，再绘制文本，从而制作文本水印等等。也可以加上更多复杂的编程控制逻辑，绘制更复杂的图片。如绘制从本地文件读取出来的图片，请参阅 [WPF 通过 DrawingContext DrawImage 绘制图片](https://blog.lindexi.com/post/WPF-%E9%80%9A%E8%BF%87-DrawingContext-DrawImage-%E7%BB%98%E5%88%B6%E5%9B%BE%E7%89%87.html ) 以及更复杂的绘制逻辑请看 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

完成绘制之后，可使用 RenderTargetBitmap 与 BitmapEncoder 配合，将绘制的内容写入到本地图片文件里面

创建 RenderTargetBitmap 对象，给定尺寸，代码如下

```csharp
    // 画布大小
    var drawingBounds = drawingVisual.Drawing.Bounds;
    // 修改为固定的尺寸
    drawingBounds = new Rect(0, 0, 1024, 768);
    var renderTargetBitmap = new RenderTargetBitmap((int) drawingBounds.Width, (int) drawingBounds.Height, 96, 96, PixelFormats.Pbgra32);
```

以上代码里面的画布大小有多个选项，一个就是使用绘制的范围，绘制的范围里面可能控制性比较差，因为会受到本身绘制的影响，可能存在绘制范围超过了预期画布尺寸，导致了输出的图片偏差。设置固定的尺寸大小，可以固定画布的尺寸，相对来说比较常用

以上代码里面的 RenderTargetBitmap 的 96 指的是图片的 DPI 值，默认情况下给 96 是比较符合视觉预期的。这里的 DPI 不是屏幕的 DPI 值，一般不应该根据当前的显示设备的 DPI 值赋值给到图片上，应该一直保持是 96 的 DPI 值。这部分有一些前置的知识，更多还请自行查阅 DPI 相关知识

将刚才绘制完成的 DrawingVisual 在 RenderTargetBitmap 渲染出来，代码如下

```csharp
    renderTargetBitmap.Render(drawingVisual);
```

现在已经拿到了 RenderTargetBitmap 对象，由于 RenderTargetBitmap 类型继承自 BitmapSource 类型，因此这里可以认为已经拿到了 BitmapSource 对象

现在的 BitmapSource 对象还在内存里面，想要将 BitmapSource 存放为图片文件需要经过图片编码步骤，将编码之后的数据写入到文件才能完成保存图片文件的保存

图片编码可使用 BitmapEncoder 类型的对象进行编码，常用的继承自 BitmapEncoder 的 PngBitmapEncoder 或 JpegBitmapEncoder 等等类型。本文这里通过 PngBitmapEncoder 编码出 png 文件，代码如下

```csharp
    var pngBitmapEncoder = new PngBitmapEncoder();
    pngBitmapEncoder.Frames.Add(BitmapFrame.Create(renderTargetBitmap));

    var file = "1.png";

    using (var fileStream = File.Create(file))
    {
        pngBitmapEncoder.Save(fileStream);
    }
```

通过以上方法即可将图片编码保存到 1.png 文件

使用本文以上的方法即可简单从控制台开始，使用 WPF 辅助绘制内容，将绘制的内容编码为图片文件保存到本地文件

本文以上代码都写在一个 Program.cs 文件里，代码非常简单

```csharp
using System.IO;
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;
var thread = new Thread(() =>
{
    var application = new Application();
    application.Startup += Application_Startup;
    application.Run();
});

void Application_Startup(object sender, StartupEventArgs e)
{
    var drawingVisual = new DrawingVisual();
    using (var drawingContext = drawingVisual.RenderOpen())
    {
        drawingContext.DrawRectangle(Brushes.Black, pen: null, new Rect(0, 0, 1024, 768));
    }

    // 画布大小
    var drawingBounds = drawingVisual.Drawing.Bounds;
    // 修改为固定的尺寸
    drawingBounds = new Rect(0, 0, 1024, 768);
    var renderTargetBitmap = new RenderTargetBitmap((int) drawingBounds.Width, (int) drawingBounds.Height, 96, 96, PixelFormats.Pbgra32);
    renderTargetBitmap.Render(drawingVisual);

    var pngBitmapEncoder = new PngBitmapEncoder();
    pngBitmapEncoder.Frames.Add(BitmapFrame.Create(renderTargetBitmap));

    var file = "1.png";

    using (var fileStream = File.Create(file))
    {
        pngBitmapEncoder.Save(fileStream);
    }
}

thread.SetApartmentState(ApartmentState.STA);
thread.Start();
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1b8661bdbea21fbef432856ddc26999505144ba2/BayheelearchachearJobarhearne) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1b8661bdbea21fbef432856ddc26999505144ba2/BayheelearchachearJobarhearne) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1b8661bdbea21fbef432856ddc26999505144ba2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1b8661bdbea21fbef432856ddc26999505144ba2
```

获取代码之后，进入 BayheelearchachearJobarhearne 文件夹，即可获取到源代码

更多 WPF 基础绘图请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

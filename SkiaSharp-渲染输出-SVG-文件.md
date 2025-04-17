
# SkiaSharp 渲染输出 SVG 文件

谷歌的 Skia 的一个卖点就是提供了完美的 SVG 的支持，包括输入和输出。输入指的是给一张 SVG 图片，将这个 SVG 渲染出来。输出就是将输出画面保存为 SVG 格式的图片。自然 SkiaSharp 是 Skia 的封装，也就带上了此功能。本文将告诉大家如何在 SkiaSharp 里面设置画面输出为 SVG 图片，使用 SkiaSharp 制作和编辑 SVG 图片

<!--more-->


<!-- CreateTime:2022/6/29 15:03:53 -->


<!-- 标签：MAUI,MauiGraphics,Skia,SkiaSharp,渲染 -->
<!-- 发布 -->

如 [dotnet 控制台 使用 Microsoft.Maui.Graphics 配合 Skia 进行绘图入门](https://blog.lindexi.com/post/dotnet-%E6%8E%A7%E5%88%B6%E5%8F%B0-%E4%BD%BF%E7%94%A8-Microsoft.Maui.Graphics-%E9%85%8D%E5%90%88-Skia-%E8%BF%9B%E8%A1%8C%E7%BB%98%E5%9B%BE%E5%85%A5%E9%97%A8.html ) 提供的方法，先新建项目安装必要的库

通过 SKSvgCanvas 提供的 SVG 画板功能进行绘制逻辑，所谓制作和编辑 SVG 图片其实就是在画板里面进行绘制，如对原有的 SVG 图片的裁剪就是画出裁剪的图片，接着保存画面。因此的核心逻辑就是将画布的渲染内容保存为 SVG 图片

创建 SKSvgCanvas 的方法十分简单，需要两个参数，分别是 SVG 的范围和输出的内容，如以下代码

```csharp
var fileName = $"xx.svg";

using var stream = File.OpenWrite(fileName);
using var skCanvas = SKSvgCanvas.Create(new SKRect(0, 0, 100, 100), stream);
```

拿到的 `skCanvas` 变量可以继续赋值给 MauiGraphics 的 SkiaCanvas 画板，用于在 MAUI 层做抽象的绘制逻辑

```csharp
var skiaCanvas = new SkiaCanvas();
skiaCanvas.Canvas = skCanvas;
```

将 SkiaCanvas 转换为 ICanvas 接口的对象，即可在后续屏蔽对 Skia 细节的处理，让绘制的逻辑都采用通用的 MAUI 逻辑

以下进行简单的绘制

```csharp
ICanvas canvas = skiaCanvas;

canvas.StrokeSize = 2;
canvas.StrokeColor = Colors.Blue;

canvas.DrawLine(10, 10, 100, 10);
```

在完成绘制之后，顺带调用一下 SKSvgCanvas 的 Flush 方法，将 SVG 内容进行输出

```csharp
skCanvas.Flush();
```

所有的代码如下

```csharp
using Microsoft.Maui.Graphics;
using Microsoft.Maui.Graphics.Skia;

using SkiaSharp;

var skImageInfo = new SKImageInfo(1920, 1080, SKColorType.Bgra8888, SKAlphaType.Opaque, SKColorSpace.CreateSrgb());

var fileName = $"xx.svg";

using var stream = File.OpenWrite(fileName);
using var skCanvas = SKSvgCanvas.Create(new SKRect(0, 0, 100, 100), stream);

var skiaCanvas = new SkiaCanvas();
skiaCanvas.Canvas = skCanvas;

ICanvas canvas = skiaCanvas;

canvas.StrokeSize = 2;
canvas.StrokeColor = Colors.Blue;

canvas.DrawLine(10, 10, 100, 10);

skCanvas.Flush();
```

可以看到输出的 svg 如以下内容

```xml
<?xml version="1.0" encoding="utf-8" ?>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100" height="100">
	<path fill="none" stroke="blue" stroke-width="2" stroke-miterlimit="10" d="M10 10L100 10"/>
</svg>
```

更多的 SkiaSharp 相关博客，还请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

本文的例子放在[github](https://github.com/lindexi/lindexi_gd/tree/bd5090f7cd66b1017a1f3a1710a3f03c03a1aafa/SkiaSharp/BihuwelcairkiDelalurnere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bd5090f7cd66b1017a1f3a1710a3f03c03a1aafa/SkiaSharp/BihuwelcairkiDelalurnere) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bd5090f7cd66b1017a1f3a1710a3f03c03a1aafa
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bd5090f7cd66b1017a1f3a1710a3f03c03a1aafa
```

获取代码之后，进入 `SkiaSharp\BihuwelcairkiDelalurnere` 文件夹

我建立了一个 SkiaSharp 的群： 788018852 欢迎大家加入讨论




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
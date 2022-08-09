# WPF 使用 Skia 解析绘制 SVG 图片

本文告诉大家如何在 WPF 里面，使用 Skia 解析绘制 SVG 图片。本文也适合控制台使用 SkiaSharp 解析绘制 SVG 图片，本文的 WPF 部分只是在 Skia 绘制完成之后，将 Skia 的内容绘制到 WPF 的 WriteableBitmap 图片，从而在界面显示

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

使用 Skia 可以很完美输出 SVG 图片作为绘制的输出。然而从 2011 开始，就有开发者在 Google 的论坛里问大佬们，是否 Skia 可以自己带上 SVG 的解析，支持传入 SVG 作为图片进行绘制。谷歌的回答是很快就会加入 SVG 导入的支持，然而现在是 2022 了，依然还没有此功能。既然 Skia 没有这个功能，那也不能要求对 Skia 的封装 SkiaSharp 有这个功能吧，如 [Matthew Leibowitz](https://github.com/mattleibow) 大佬的回复

<!-- ![](image/WPF 使用 Skia 解析绘制 SVG 图片/WPF 使用 Skia 解析绘制 SVG 图片0.png) -->

![](http://image.acmx.xyz/lindexi%2F2022881931555567.jpg)

详细请看 [https://github.com/mono/SkiaSharp.Extended/issues/87#issuecomment-552113673](https://github.com/mono/SkiaSharp.Extended/issues/87#issuecomment-552113673)

在上面的这个帖子也介绍了两个 SVG 解析库，其中一个就是我用过的 [SVG.NET](https://github.com/svg-net/SVG) 库，可惜这个库不是 Skia 专用的，本文也就不介绍他。另一个库是 [Svg.Skia](https://github.com/wieslawsoltes/Svg.Skia ) 库，这是给 Skia 专用的库

接下来咱将使用这个 [Svg.Skia](https://github.com/wieslawsoltes/Svg.Skia ) 库，在 WPF 应用里，加载 SVG 文件，使用 Skia 渲染

按照惯例的第一步就是安装 NuGet 包，通过 NuGet 安装 Svg.Skia 库，或者编辑 csproj 项目文件加上以下代码

```xml
  <ItemGroup>
    <PackageReference Include="Svg.Skia" Version="0.5.16" />
  </ItemGroup>
```

开始写代码之前，引用命名空间

```csharp
using SkiaSharp;
using Svg.Skia;
```

解析 SVG 的方法是通过 SKSvg 类型进行加载，转换为 Skia 的 SKPicture 对象，代码如下

```csharp
     using var skSvg = new SKSvg();
     skSvg.Load(svgFile);
     if (skSvg.Picture is null)
     {
         return;
     }

     var skSvgPicture = skSvg.Picture;
```

以上的 SKSvg 就是 [Svg.Skia](https://github.com/wieslawsoltes/Svg.Skia ) 提供的类型

为了方便进行渲染，获取到 SVG 的尺寸，先转换为 SKBitmap 类型。这里的设计是转换失败返回空，以上的方法是不能支持所有的 SVG 格式的文件的，只对 SVG 1.1 版本支持比较好

```csharp
 var skBitmap = skSvgPicture.ToBitmap(SKColor.Empty, 1, 1, SKColorType.Bgra8888, SKAlphaType.Premul, SKColorSpace.CreateSrgb());
```

如以上的代码，传入足够的参数，转换为 SKBitmap 类型。参数基本上就是约定了像素数据的表示和透明度支持

拿到 SKBitmap 对象，再根据 [WPF 使用 Skia 绘制 WriteableBitmap 图片](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Skia-%E7%BB%98%E5%88%B6-WriteableBitmap-%E5%9B%BE%E7%89%87.html ) 提供的方法进行绘制

```csharp
 var writeableBitmap = new WriteableBitmap(skBitmap.Width, skBitmap.Height, 96, 96, PixelFormats.Bgra32,
     BitmapPalettes.Halftone256Transparent);

 var skImageInfo = new SKImageInfo()
 {
     Width = skBitmap.Width,
     Height = skBitmap.Height,
     ColorType = SKColorType.Bgra8888,
     AlphaType = SKAlphaType.Premul,
     ColorSpace = SKColorSpace.CreateSrgb()
 };

 using SKSurface surface = SKSurface.Create(skImageInfo, writeableBitmap.BackBuffer);

 writeableBitmap.Lock();
 surface.Canvas.DrawBitmap(skBitmap,0,0);

 writeableBitmap.AddDirtyRect(new Int32Rect(0,0, skBitmap.Width, skBitmap.Height));
 writeableBitmap.Unlock();

 var image = new Image()
 {
     Width = skBitmap.Width,
     Height = skBitmap.Height,
     Source = writeableBitmap,
 };

 Root.Children.Add(image);
```

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/23259e0ffda16851834d757c0b1619dee299c7c7/GafawwaybalachaCemleardearha) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/23259e0ffda16851834d757c0b1619dee299c7c7/GafawwaybalachaCemleardearha) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 23259e0ffda16851834d757c0b1619dee299c7c7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 GafawwaybalachaCemleardearha 文件夹

# WPF 使用 Skia 绘制 WriteableBitmap 图片

本文告诉大家如何在 WPF 中使用 SkiaSharp 调用 Skia 这个全平台底层渲染框架，使用绘制命令在 WriteableBitmap 图片上绘制内容

<!--more-->
<!-- 发布 -->

谷歌提出了 Skia 全平台渲染框架，这是一个很底层的框架，详细请看 [google/skia: Skia is a complete 2D graphic library for drawing Text, Geometries, and Images.](https://github.com/google/skia)

而 SkiaSharp 是 mono 组织对 Skia 的 .NET 封装库，可以完全用到 Skia 的底层渲染能力，详细请看 [mono/SkiaSharp: SkiaSharp is a cross-platform 2D graphics API for .NET platforms based on Google's Skia Graphics Library. It provides a comprehensive 2D API that can be used across mobile, server and desktop models to render images.](https://github.com/mono/SkiaSharp/)

那么如何在 WPF 使用 SkiaSharp 绘制出 WriteableBitmap 在 WPF 中使用？其实 WriteableBitmap 是将一个数组里面的像素在屏幕显示，而 SKSurface 可以从一个像素数组开始创建，创建的时候需要规定这个数组对应的图片的格式，包括图片的大小以及 RGB 像素格式

使用下面代码创建一个简单的界面，在这个界面里面点击按钮将会给 Image 控件赋值使用 Skia 创建的图片

```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>
        <Image x:Name="Image" Margin="10,10,10,10"></Image>
        <Button Margin="10,10,10,10" Grid.Row="1" Content="使用Skia绘制" Click="Button_OnClick"></Button>
    </Grid>
```

在 WPF 里面创建一个 WriteableBitmap 可以使用如下代码

```csharp
        private WriteableBitmap CreateImage(int width, int height)
        {
            var writeableBitmap = new WriteableBitmap(width, height, 96, 96, PixelFormats.Bgra32, BitmapPalettes.Halftone256Transparent);
            return writeableBitmap;
        }
```

为什么需要创建一个方法来创建，原因是参数 PixelFormats.Bgra32 和 BitmapPalettes.Halftone256Transparent 将会和后续的 Skia 创建相关

在 Skia 里面和 D2D 一样有 Surface 的概念，也就是可以将绘制命令输入到 Skia 绘制到 Surface 上，而绘制内容将会作为像素数组放在传入的数组里面

小伙伴是否还记得 [WPF 使用不安全代码快速从数组转 WriteableBitmap](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%AE%89%E5%85%A8%E4%BB%A3%E7%A0%81%E5%BF%AB%E9%80%9F%E4%BB%8E%E6%95%B0%E7%BB%84%E8%BD%AC-WriteableBitmap.html) 的方法，其实 Skia 在 WriteableBitmap 绘制的本质就是这样

在开始绘制之前需要调用 WriteableBitmap 的 Lock 方法，接着在绘制完成之后，需要调用 AddDirtyRect 和 Unlock 方法

大概的绘制代码如下

```csharp
        private void UpdateImage(WriteableBitmap writeableBitmap)
        {
            int width = (int)writeableBitmap.Width,
                height = (int)writeableBitmap.Height;
            writeableBitmap.Lock();
            var skImageInfo = new SKImageInfo()
            {
                Width = width,
                Height = height,
                ColorType = SKColorType.Bgra8888,
                AlphaType = SKAlphaType.Premul,
                ColorSpace = SKColorSpace.CreateSrgb()
            };

            using (var surface = SKSurface.Create(skImageInfo, writeableBitmap.BackBuffer))
            {
                SKCanvas canvas = surface.Canvas;
                canvas.Clear(new SKColor(130, 130, 130));
                canvas.DrawText("SkiaSharp on Wpf!", 50, 200, new SKPaint() { Color = new SKColor(0, 0, 0), TextSize = 100 });
                canvas.DrawText("https://blog.lindexi.com", new SKPoint(50, 500), new SKPaint(new SKFont(SKTypeface.FromFamilyName("微软雅黑")))
                {
                    Color = new SKColor(0, 0, 0),
                    TextSize = 20
                });
            }
            writeableBitmap.AddDirtyRect(new Int32Rect(0, 0, width, height));
            writeableBitmap.Unlock();
        }
```

绘制界面大概如下

![](http://image.acmx.xyz/lindexi%2F20208301919281724.jpg)

本文的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e414c652ae503d882ad13eb844e70f8bc787f287/ReewheaberekaiNayweelehe) 欢迎小伙伴访问

更多使用方法还需要小伙伴自己去玩

当前可以使用 SkiaSharp 支持 Window 端和 Linux 端以及 macOS 和 iOS 和安卓端的绘制，其中 Xamarin 中集成 Skia 的成熟度是最高的

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

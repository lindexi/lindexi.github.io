# WPF 对接 Vortice 在 Direct2D 绘制从 WIC 加载的图片

本文告诉大家如何通过 Vortice 在 Direct2D 里面绘制图片，图片的来源是 WIC 加载出的图片

<!--more-->

<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

在上一篇博客告诉了大家如何对接 Vortice 调用 WIC 加载图片，上一篇博客是将 WIC 层创建的 IWICBitmap 图片放入到 WPF 层进行渲染。本文将告诉大家如何在 Direct2D 里将 WIC 加载的图片绘制

核心的两个点就是用拿到的 IWICBitmapFrameDecode 进行 IWICFormatConverter 转换图片格式，转换为 Format32bppPBGRA 对 Direct2D 友好的格式，再通过 CreateBitmapFromWicBitmap 方法转换为 ID2D1Bitmap 加入绘制

通过上一篇博客可以了解到如下代码可以加载本地图片文件到 WIC 层进行解码

```csharp
        using var wicImagingFactory = new IWICImagingFactory();
        var imageFilePath = System.IO.Path.GetFullPath("Image.png");
        using var wicStream = wicImagingFactory.CreateStream(imageFilePath, FileAccess.Read);
        using var decoder = wicImagingFactory.CreateDecoderFromStream(wicStream, DecodeOptions.CacheOnLoad/*参数和 WPF 一样*/);
        // 解码器将可以解码出图片，对于动态图片可以解析出多张图片出来，对于静态图片只能解析出一张
        // 对于静态图片（区别于 gif 等动态图片）只须取首个
        using IWICBitmapFrameDecode imageFrame = decoder.GetFrame(0);
```

这时拿到的 `imageFrame` 对象还不能立刻扔到 `ID2D1RenderTarget.CreateBitmapFromWicBitmap` 里面，因为图片的像素格式不一定是对 D2D 友好的，如果没有经过图片颜色格式转换，也许就炸掉，提示 0x88982F80 错误，即 WINCODEC_ERR_UNSUPPORTEDPIXELFORMAT 格式不受支持

转换图片格式自然是不需要大家手写许多代码的，也不需要去担心在 CPU 或 GPU 转码的性能问题，通过内置的 IWICFormatConverter 进行转换，具体转换细节可以作为黑盒的存在，根据不同的硬件设备和驱动条件决定是否走硬件加速。当然，自己想不开，手动转码也是可以的，本文还是写给想得开的伙伴看的

先创建出来 IWICFormatConverter 格式转换器，代码如下

```csharp
        // 图片的格式不一定是能符合 D2D 预期的，转换一下格式
        // 否则 CreateBitmapFromWicBitmap 失败 0x88982F80 WINCODEC_ERR_UNSUPPORTEDPIXELFORMAT
        using IWICFormatConverter converter = wicImagingFactory.CreateFormatConverter();
```

接着调用 Initialize 方法进行初始化，这个 IWICFormatConverter 类型从设计上是继承 IWICBitmapSource 的，也就是这里其实没有立刻做转换，而是表示一个状态，具体在哪一层做实际的转换，这是封装起来的黑盒

```csharp
        // 这里不是真实的立刻进行转换哦，实际转换执行是隐藏起来的
        converter.Initialize(imageFrame, Vortice.WIC.PixelFormat.Format32bppPBGRA, BitmapDitherType.None, null, 0, BitmapPaletteType.MedianCut);
        // 这个 IWICFormatConverter 也继承是 IWICBitmapSource 类型
```

完成转换器逻辑，即可将转换器扔到 CreateBitmapFromWicBitmap 里创建出 `D2D.ID2D1Bitmap` 进行绘制

```csharp
        D2D.ID2D1Bitmap d2DBitmap = renderTarget.CreateBitmapFromWicBitmap(converter);

        renderTarget.DrawBitmap(d2DBitmap);
```

以下是打开本地文件，进行解码，转换颜色格式，创建 ID2D1Bitmap 绘制的代码

```csharp
    private static void Render(D2D.ID2D1RenderTarget renderTarget)
    {
        using var wicImagingFactory = new IWICImagingFactory();
        var imageFilePath = System.IO.Path.GetFullPath("Image.png");
        using var wicStream = wicImagingFactory.CreateStream(imageFilePath, FileAccess.Read);
        using var decoder = wicImagingFactory.CreateDecoderFromStream(wicStream, DecodeOptions.CacheOnLoad/*参数和 WPF 一样*/);
        // 解码器将可以解码出图片，对于动态图片可以解析出多张图片出来，对于静态图片只能解析出一张
        // 对于静态图片（区别于 gif 等动态图片）只须取首个
        using IWICBitmapFrameDecode imageFrame = decoder.GetFrame(0);

        using IWICBitmap bitmap = wicImagingFactory.CreateBitmapFromSource(imageFrame, BitmapCreateCacheOption.CacheOnLoad);

        // 图片的格式不一定是能符合 D2D 预期的，转换一下格式
        // 否则 CreateBitmapFromWicBitmap 失败 0x88982F80 WINCODEC_ERR_UNSUPPORTEDPIXELFORMAT
        using IWICFormatConverter converter = wicImagingFactory.CreateFormatConverter();
        // 这里不是真实的立刻进行转换哦，实际转换执行是隐藏起来的
        converter.Initialize(imageFrame, Vortice.WIC.PixelFormat.Format32bppPBGRA, BitmapDitherType.None, null, 0, BitmapPaletteType.MedianCut);
        // 这个 IWICFormatConverter 也继承是 IWICBitmapSource 类型

        D2D.ID2D1Bitmap d2DBitmap = renderTarget.CreateBitmapFromWicBitmap(converter);

        renderTarget.DrawBitmap(d2DBitmap);
    }
```

如果要将图片绘制在给定的范围呢？可以通过将图片转换为贴图画刷的方式然后通过矩形或其他几何承载，如以下的代码将图片绘制在矩形上，通过矩形控制绘制在哪个范围

```csharp
        using D2D.ID2D1Bitmap d2DBitmap = renderTarget.CreateBitmapFromWicBitmap(converter);

        using var brush = renderTarget.CreateBitmapBrush(d2DBitmap);

        renderTarget.FillRectangle(new Vortice.RawRectF(10, 10, 900, 600), brush);
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/1e2b04cdfd620ec666e6dbcf58b561dae575e9c1/WpfVorticeWicTest) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1e2b04cdfd620ec666e6dbcf58b561dae575e9c1/WpfVorticeWicTest) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1e2b04cdfd620ec666e6dbcf58b561dae575e9c1
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1e2b04cdfd620ec666e6dbcf58b561dae575e9c1
```

获取代码之后，进入 WpfVorticeWicTest 文件夹

更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

另外，我创建了专门聊 Vortice 的 QQ 群： 622808968 欢迎加入交流技术
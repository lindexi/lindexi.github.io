# WPF 对接 Vortice 调用 D2D 使用 IWICBitmap 离屏渲染

通过 Vortice 库可以使用非常底层的方式调用到 Direct2D1 进行渲染，本文将使用 D2D 离屏渲染到 IWICBitmap 上，再使用一点点反射黑科技，直接将此 IWICBitmap 对接到 WPF 框架里。本文提供的这个方法可以实现极高性能且只有很少的转换损耗的离屏渲染方式，唯一的一个缺点是需要进行一点反射调用，适合用来静态画面渲染上

<!--more-->

<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

在 WPF 的渲染底层里，对于图片来说，都是采用 WIC Bitmap 参与渲染。而刚好 Direct2D1 可以从一个 IWICBitmap 上使用 CreateWicBitmapRenderTarget 方法创建 ID2D1RenderTarget 对象，在 ID2D1RenderTarget 上执行绘制指导命令，从而实现将画面绘制到 IWICBitmap 上

于是新建出一个 IWICBitmap 对象，接着挂上 D2D 的 ID2D1RenderTarget 进行绘制。完成之后，将 IWICBitmap 封装为一个 BitmapSource 对象，扔给 WPF 层，当成图片接入 WPF 的渲染框架

创建 IWICBitmap 对象和挂上 D2D 以及绘制逻辑的细节，请参阅 [dotnet C# 使用 Vortice 支持 Direct2D1 离屏渲染](https://blog.lindexi.com/post/dotnet-C-%E4%BD%BF%E7%94%A8-Vortice-%E6%94%AF%E6%8C%81-Direct2D1-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html )

以下是本文使用的代码，这里就不展开细节

```csharp
    private static Task<IWICBitmap> OffScreenRenderingWICBitmapAsync()
    {
        return Task.Run(OffScreenRenderingWICBitmap);
    }
    private static IWICBitmap OffScreenRenderingWICBitmap()
    {
        using var wicImagingFactory = new IWICImagingFactory();
        IWICBitmap wicBitmap =
            wicImagingFactory.CreateBitmap(1000, 1000, Win32.Graphics.Imaging.Apis.GUID_WICPixelFormat32bppPBGRA);

        using D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();
        var renderTargetProperties = new D2D.RenderTargetProperties(PixelFormat.Premultiplied);
        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory.CreateWicBitmapRenderTarget(wicBitmap, renderTargetProperties);

        using var renderTarget = d2D1RenderTarget;
        // 开始绘制逻辑
        renderTarget.BeginDraw();

        Render(renderTarget);

        renderTarget.EndDraw();

        return wicBitmap;
    }

    private static void Render(D2D.ID2D1RenderTarget renderTarget)
    {
        // 以下是测试代码
        // 假装是耗时的渲染
        var color = new Color4((byte)Random.Shared.Next(255), (byte)Random.Shared.Next(255),
            (byte)Random.Shared.Next(255));
        renderTarget.Clear(color);

        color = new Color4((byte)Random.Shared.Next(255), (byte)Random.Shared.Next(255),
            (byte)Random.Shared.Next(255));
        using var brush = renderTarget.CreateSolidColorBrush(color);

        // 10万个圆，无论是啥都顶不住
        for (int i = 0; i < 100000; i++)
        {
            renderTarget.DrawEllipse(new D2D.Ellipse(new Vector2(Random.Shared.Next(900), Random.Shared.Next(900)),Random.Shared.Next(1,5), Random.Shared.Next(1, 5)),brush,Random.Shared.Next(1,2));
        }
    }
```

从上面代码可以看到，这是完全在另一个线程执行的逻辑，如此将不会卡住主线程。可以放心在后台线程里，执行复杂的逻辑，绘制一张有趣的画面。例如本文就采用啥都顶不住的画 10 万个圆的方法

完成离屏渲染之后，需要将 IWICBitmap 的结果对接到 WPF 框架，对接方法是封装为一个 BitmapSource 对象。可以 WPF 框架里面没有对外公开的 UnmanagedBitmapWrapper 类型，只是使用没有公开的类型就需要用到一点点反射

```csharp
    private static BitmapSource WICBitmapToBitmapSource(IWICBitmap wicBitmap)
    {
        var presentationCoreAssembly = typeof(BitmapSource).Assembly;
        var bitmapSourceSafeMILHandleType =
            presentationCoreAssembly.GetType("System.Windows.Media.Imaging.BitmapSourceSafeMILHandle", throwOnError: true)!;
        var bitmapSourceSafeMILHandleConstructor =
            bitmapSourceSafeMILHandleType.GetConstructor(BindingFlags.NonPublic | BindingFlags.Instance,
                new Type[] { typeof(IntPtr) })!;

        var bitmapSourceSafeMILHandle =
            bitmapSourceSafeMILHandleConstructor.Invoke(new object[] { wicBitmap.NativePointer });

        var unmanagedBitmapWrapperType =
            presentationCoreAssembly.GetType("System.Windows.Media.Imaging.UnmanagedBitmapWrapper")!;

        var unmanagedBitmapWrapperConstructor =
            unmanagedBitmapWrapperType.GetConstructor(BindingFlags.Public | BindingFlags.Instance,
                new Type[] { bitmapSourceSafeMILHandleType })!;

        var unmanagedBitmapWrapper = unmanagedBitmapWrapperConstructor.Invoke(new object[] { bitmapSourceSafeMILHandle });
        return (BitmapSource) unmanagedBitmapWrapper;
    }
```

按照 WPF 框架的源代码，可以看到 UnmanagedBitmapWrapper 的定义和构造函数如下

```csharp
    internal sealed class UnmanagedBitmapWrapper : BitmapSource
    {
        public UnmanagedBitmapWrapper(BitmapSourceSafeMILHandle bitmapSource) :
            base(true)
        {            
            _bitmapInit.BeginInit();

            //
            // This constructor is used by BitmapDecoder and BitmapFrameDecode for thumbnails and
            // previews. The bitmapSource parameter comes from BitmapSource.CreateCachedBitmap
            // which already calculated memory pressure, so there's no need to do it here.
            //
            WicSourceHandle = bitmapSource;
            _bitmapInit.EndInit();
            UpdateCachedSettings();
        }
        // 忽略代码
    }
```

原本这个 UnmanagedBitmapWrapper 是设计给解码器等使用的，不是公开使用的，但是我看起来这个类型很清真，也许开放出来也是可以的

在拿到继承 BitmapSource 的 UnmanagedBitmapWrapper 对象之后，即可作为某个图片的 Source 使用

```csharp
            Image.Source = unmanagedBitmapWrapper;
```

为了方便演示效果，在 WPF 的 MainWindow 放一个 Image 控件，如下面代码

```xml
        <Image x:Name="Image"></Image>
```

接着在 Loaded 事件之后，先异步在后台线程调用 D2D 的渲染，将渲染结果封装为 BitmapSource 再设置给图片

```csharp
    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        try
        {
            using IWICBitmap wicBitmap = await OffScreenRenderingWICBitmapAsync();

            var unmanagedBitmapWrapper = WICBitmapToBitmapSource(wicBitmap);

            Image.Source = unmanagedBitmapWrapper;
        }
        catch (Exception)
        {
            // 这里是 async void 线程的顶层，如果有任何异常，那应用就炸了
            // 而采用离屏渲染的 OffScreenRenderingWICBitmapAsync 是预期会有很多奇怪的异常
        }
    }
```

试试跑跑上本文的例子，可以从文本末尾获取到项目的全部代码的可构建运行项目，测试一下对 WPF 的影响。预计此方法对 WPF 的影响是非常小的，损耗约等于渲染一张图，而且还是一张不需要解码的图片的损耗

既然这个方式需要这么好用，那再用反射似乎也说不过去，刚好我是 WPF 框架的开发者，我在想着要不要将这个 UnmanagedBitmapWrapper 类型开放好了

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/1de394636dc15865ab90301230ed7ce37fe01ca0/WpfVorticeWicTest) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1de394636dc15865ab90301230ed7ce37fe01ca0/WpfVorticeWicTest) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1de394636dc15865ab90301230ed7ce37fe01ca0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1de394636dc15865ab90301230ed7ce37fe01ca0
```

获取代码之后，进入 WpfVorticeWicTest 文件夹
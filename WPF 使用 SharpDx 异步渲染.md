# WPF 使用 SharpDx 异步渲染

本文告诉大家如何通过 SharpDx 进行异步渲染，但是因为在 WPF 是需要使用 D3DImage 画出来，所以渲染只是画出图片，最后的显示还是需要 WPF 在他自己的主线程渲染。

<!--more-->
<!-- CreateTime:2019/10/23 21:18:38 -->

<div id="toc"></div>
<!-- 标签：WPF,D2D,DirectX,SharpDX,渲染 -->

本文是一个系列，希望大家从第一篇开始看

 - [WPF 使用 Direct2D1 画图入门](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html )

 - [WPF 使用 Direct2D1 画图 绘制基本图形](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE-%E7%BB%98%E5%88%B6%E5%9F%BA%E6%9C%AC%E5%9B%BE%E5%BD%A2.html )

 - [WPF 使用 SharpDX](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX.html )

 - [WPF 使用 SharpDX 在 D3DImage 显示](https://lindexi.gitee.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX-%E5%9C%A8-D3DImage-%E6%98%BE%E7%A4%BA.html ) 

 - [WPF 使用封装的 SharpDx 控件](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )

 - [WPF 使用 SharpDx 异步渲染](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E5%BC%82%E6%AD%A5%E6%B8%B2%E6%9F%93.html )

更多请看 [WPF 使用 SharpDx 渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

虽然上一篇告诉大家如何使用封装的 SharpDx 控件，但是大家也看到了核心是使用`CompositionTarget`告诉刷新的。

这个方法适合不停变化的控件，如果是很少刷新的控件使用这个方法会降低 WPF 的性能。

因为 CompositionTarget 刷新数太快了，而且每次都需要重复刷新一个图片，显示的性能比不过自带的控件。

## 使用方法

因为使用 SharpDx 在 WPF 除了使用 D3DImage 还可以使用 D3D11Image 但是这个需要分开 x86 和 x64 。现在使用的方法是把 D3DImage 作为图片画出来，如果使用 D3D11Image 也没有什么性能提升。

所以本文就和[WPF 使用封装的 SharpDx 控件](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )使用的基类不同，原来的基类是 Image 现在的基类是 `FrameworkElement` 。但是如果使用 Image 而且每次刷新的都是比较小的，性能会比使用 FrameworkElement 画出高一些。

这里因为封装没有告诉需要刷新的大小，所以只能每次都全部刷新，这样的性能使用 FrameworkElement 不会降低。

下面创建一个类，继承 SharpDxMaynumaSejair ，这个 SharpDxMaynumaSejair 是继承 FrameworkElement 而不是图片，这个类的代码放在文章最后，使用这个类可以异步渲染。


```csharp
    public abstract class SharpDxMaynumaSejair : FrameworkElement

```

请随意写一个类继承 SharpDxMaynumaSejair 并且添加重写的 OnRender 函数。

下面是 SharpDxMaynumaSejair 类的 OnRender 方法，通过继承他就可以使用 SharpDx 画出来。

```csharp
protected abstract void OnRender(SharpDX.Direct2D1.RenderTarget renderTarget);
```

其他的代码和[WPF 使用封装的 SharpDx 控件](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )使用的差不多

直接通过 OnRender 就可以进行渲染，但是 OnRender 是被触发的，触发的方法是调用基类 `Rendering` 函数，调用了这个函数会进入异步的 SharpDx 渲染，渲染完成再通过 WPF 渲染画出来。

因为不需要使用 CompositionTarget.Rendering 渲染，所以可以提高 WPF 刷新速度。

这个类可以在执行渲染计算复杂使用，假如需要渲染出 10000 个椭圆，而且有很多重叠，而且不需要立刻渲染。那么就可以使用本文的这个类，这个类可以在调用时异步渲染，不会卡 UI 线程，在 SharpDx 渲染完成再通过 WPF 渲染，这时 WPF 渲染也就是画出图片，性能比画出 10000 个椭圆快很多。通过这个方法可以提高渲染性能，提高软件打开的性能。

但是通过这个方法建议软件是 x64 因为需要很多内存。

下面来告诉大家本文这个类的原理。

## 绑定

如果需要使用 SharpDx 需要把 SharpDX.Direct3D11 和 D3DImage 绑定，调用时不能在这个控件的 Load 前，不然无法拿到大小。

下面这个方法和[WPF 使用封装的 SharpDx 控件](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )使用相同，所以我就直接写代码不解释了。

```csharp
        private void CreateAndBindTargets(int actualWidth, int actualHeight)
        {
            var width = Math.Max(actualWidth, 100);
            var height = Math.Max(actualHeight, 100);

            var renderDesc = new SharpDX.Direct3D11.Texture2DDescription
            {
                BindFlags = SharpDX.Direct3D11.BindFlags.RenderTarget | SharpDX.Direct3D11.BindFlags.ShaderResource,
                Format = SharpDX.DXGI.Format.B8G8R8A8_UNorm,
                Width = width,
                Height = height,
                MipLevels = 1,
                SampleDescription = new SharpDX.DXGI.SampleDescription(1, 0),
                Usage = SharpDX.Direct3D11.ResourceUsage.Default,
                OptionFlags = SharpDX.Direct3D11.ResourceOptionFlags.Shared,
                CpuAccessFlags = SharpDX.Direct3D11.CpuAccessFlags.None,
                ArraySize = 1
            };

            var device = new SharpDX.Direct3D11.Device(DriverType.Hardware,
                SharpDX.Direct3D11.DeviceCreationFlags.BgraSupport);

            _device = device;

            var renderTarget = new SharpDX.Direct3D11.Texture2D(device, renderDesc);

            var surface = renderTarget.QueryInterface<SharpDX.DXGI.Surface>();

            var d2DFactory = new SharpDX.Direct2D1.Factory();

            var renderTargetProperties =
                new SharpDX.Direct2D1.RenderTargetProperties(
                    new SharpDX.Direct2D1.PixelFormat(SharpDX.DXGI.Format.Unknown,
                        SharpDX.Direct2D1.AlphaMode.Premultiplied));

            _d2DRenderTarget = new SharpDX.Direct2D1.RenderTarget(d2DFactory, surface, renderTargetProperties);

            SetRenderTarget(renderTarget);

            device.ImmediateContext.Rasterizer.SetViewport(0, 0, width, height);

            CreationProperties creationProperties = new CreationProperties
            {
                DebugLevel = DebugLevel.Error,
                Options = DeviceContextOptions.EnableMultithreadedOptimizations,
                ThreadingMode = ThreadingMode.MultiThreaded,
            };


            _d2dContext = new DeviceContext(surface, creationProperties);

            _d2DRenderTarget = _d2dContext;

            InvalidateVisual();
        }

```

如果大家有对比两个函数，会发现有一个属性不相同，原来是 Direct3D11.Device 被我拿出一个字段，这个字段在下面会使用到。

虽然已经写好 D3DImage 但是如何显示？

继承 FrameworkElement 可以重写 OnRender 。通过 OnRender 可以画出图片，而 D3Dimage 就是 ImageSource，虽然可以看到我自己定义的也是 OnRender， 这个函数和自己定义的不相同，虽然我把自己定义的函数也是和他使用相同的命名。看到下面代码也许就知道我说的是什么。

需要注意，如果因为`_d3D.PixelWidth`为0抛出异常，那么就可能是绑定的时候在 Load 之前，需要修改一下代码。

```csharp
        protected override void OnRender(DrawingContext drawingContext)
        {
            drawingContext.DrawImage(_d3D, new Rect(new Size(_d3D.PixelWidth, _d3D.PixelHeight)));
        }
```

## 渲染为什么空白

现在已经完成了修改继承类，但是原来使用的渲染还是没有修改。如果大家尝试在一个按钮按下时，进行刷新。代码大概是在 按钮按下时调用`Rendering`函数，这个函数只执行一次

```csharp
        public void Rendering()
        {
            
            _d2dContext.BeginDraw();

            OnRender(_d2dContext);

            _d2dContext.EndDraw();


            _d3D.Lock();


            _d3D.AddDirtyRect(new Int32Rect(0, 0, _d3D.PixelWidth, _d3D.PixelHeight));

            _d3D.Unlock();

            base.InvalidateVisual();
        }
```

如果大家运行了代码，会发现现在的界面虽然有执行`OnRender`但是显示和希望的不一样。

原因是没有等待 SharpDx 画完，虽然调用了`EndDraw`但是只是把渲染命令发给显卡。

那么如何等待 SharpDx 画完

## 等待画完

如果刚才看到 CreateAndBindTargets 会看到把 Direct3D11.Device 放在字段，因为在 Rendering 就需要使用这个字段等待显卡刷新。

请看下面的代码

```csharp
        public void Rendering()
        {
            _d2dContext.BeginDraw();

            OnRender(_d2dContext);

            _d2dContext.EndDraw();

            _device.ImmediateContext.Flush();


            _d3D.Lock();


            _d3D.AddDirtyRect(new Int32Rect(0, 0, _d3D.PixelWidth, _d3D.PixelHeight));

            _d3D.Unlock();

            base.InvalidateVisual();
        }
```
 
尝试修改代码运行一下，可以如果调用了一次函数就可以刷新一次。

如果是在按钮按下是刷新，但是渲染很多内容，那么主线程会在 Flush 占用很多资源。

在 WPF 的渲染，是把主线程和渲染线程分开，经常说的主线程是没有做渲染的，在 DrawingContext 实际上不是调用了显示，而且通过 Channel 发送到Dx渲染，也就是调用函数只是告诉显卡如何渲染。

在这里也是需要做相同的方法。

## 异步渲染

大家也可以看到，只需要使用一个新的线程去等待渲染就可以，使用新线程的方法是 Task ，但是不能把 d3dImage 放在另一个线程，他必须在主线程。

修改一下代码

```csharp
        public async void Rendering()
        {

            await Task.Run(() =>
            {
                _d2dContext.BeginDraw();

                OnRender(_d2dContext);

                _d2dContext.EndDraw();

                _device.ImmediateContext.Flush();
            });


            _d3D.Lock();


            _d3D.AddDirtyRect(new Int32Rect(0, 0, _d3D.PixelWidth, _d3D.PixelHeight));

            _d3D.Unlock();

            base.InvalidateVisual();
        }

```

现在就可以在另一个线程告诉 SharpDx 如何画，然后在另一个线程等待 SharpDx 画出来。这样可以做到异步渲染。

需要告诉大家，异步渲染不是多线程渲染，原因是渲染还是需要显卡来做，如果显卡的资源有限，那么渲染需要的时间不会降低。

如果需要设置多线程渲染，可以通过在`CreationProperties`使用`ThreadingMode.MultiThreaded`现在上面的代码已经这样使用。

不过大家不要直接使用这个类，因为上面代码使用`Task.Run`，如果在线程池没有资源，那么这个代码可能会等很久，这样的性能比较差。

这个控件可以用在不需要立刻渲染的资源，但是渲染很慢，可以在用户做其他的输入进行渲染。因为默认的渲染都会让用户感觉软件速度有些慢，不过和这个做法相同的是使用 RenderTargetBitmap ，在另一个线程渲染，然后在主线程显示。

和 RenderTargetBitmap 不同的，本文的方法可以在显卡渲染，渲染性能比 RenderTargetBitmap 高。

## 多线程渲染

下面告诉大家如何使用 RenderTargetBitmap 多线程渲染

首先创建一个字段，在这个字段为空就需要调用函数创建

```csharp
        private RenderTargetBitmap _stouFa;

```

这个方法适合只有进行很少的渲染，而且很慢的应用。实际上 RenderTargetBitmap 可以 Freeze ，所以在另一个线程渲染是可以。

```csharp
        protected override void OnRender(DrawingContext drawingContext)
        {
            if (_stouFa == null)
            {
                TesuFudresel();
            }
            else
            {
                drawingContext.DrawImage(_stouFa, new Rect(new Size(ActualWidth, ActualHeight)));
            }
        }
```

在方法 TesuFudresel 就是创建渲染

```csharp
        private void TesuFudresel()
        {

            Task.Run(() =>
            {
                var renderTargetBitmap = new RenderTargetBitmap(100, 100, 96, 96, new PixelFormat());
                var drawingVisual = new DrawingVisual();
                var drawingContext = drawingVisual.RenderOpen();
                drawingContext.DrawRectangle(Brushes.Chartreuse, new Pen(Brushes.Chartreuse, 2),
                    new Rect(new Point(10, 10), new Size(100, 100)));
                drawingContext.Close();
                renderTargetBitmap.Render(drawingVisual);
                renderTargetBitmap.Freeze();

                _stouFa = renderTargetBitmap;
            });

        }
```

代码主要就是使用 drawingVisual 画出来，然后让对象可以跨线程。

本文就告诉大家如何使用 SharpDx 异步渲染，还告诉大家如何使用 WPF 自带的类进行多线程渲染，下面就是本文这个控件的代码

建议大家自己写一个线程调度而不是使用 Task ，因为最近在写 Avalon 所以暂时我也没有把下面的类写的可以在产品使用。请大家参考代码而不是在项目使用这个代码，因为存在 Task 需要等很久和代码没有优化。

```csharp
    public abstract class SharpDxMaynumaSejair : FrameworkElement
    {
        private D3DImage _d3D = new D3DImage();

        /// <inheritdoc />
        protected SharpDxMaynumaSejair()
        {
            Loaded += SharpDxMaynumaSejair_Loaded;
        }

        private void SharpDxMaynumaSejair_Loaded(object sender, RoutedEventArgs e)
        {
            CreateAndBindTargets((int) ActualWidth, (int) ActualHeight);
        }

        private void CreateAndBindTargets(int actualWidth, int actualHeight)
        {
            var width = Math.Max(actualWidth, 100);
            var height = Math.Max(actualHeight, 100);

            var renderDesc = new SharpDX.Direct3D11.Texture2DDescription
            {
                BindFlags = SharpDX.Direct3D11.BindFlags.RenderTarget | SharpDX.Direct3D11.BindFlags.ShaderResource,
                Format = SharpDX.DXGI.Format.B8G8R8A8_UNorm,
                Width = width,
                Height = height,
                MipLevels = 1,
                SampleDescription = new SharpDX.DXGI.SampleDescription(1, 0),
                Usage = SharpDX.Direct3D11.ResourceUsage.Default,
                OptionFlags = SharpDX.Direct3D11.ResourceOptionFlags.Shared,
                CpuAccessFlags = SharpDX.Direct3D11.CpuAccessFlags.None,
                ArraySize = 1
            };

            var device = new SharpDX.Direct3D11.Device(DriverType.Hardware,
                SharpDX.Direct3D11.DeviceCreationFlags.BgraSupport);

            _device = device;

            var renderTarget = new SharpDX.Direct3D11.Texture2D(device, renderDesc);

            var surface = renderTarget.QueryInterface<SharpDX.DXGI.Surface>();

            var d2DFactory = new SharpDX.Direct2D1.Factory();

            var renderTargetProperties =
                new SharpDX.Direct2D1.RenderTargetProperties(
                    new SharpDX.Direct2D1.PixelFormat(SharpDX.DXGI.Format.Unknown,
                        SharpDX.Direct2D1.AlphaMode.Premultiplied));

            _d2DRenderTarget = new SharpDX.Direct2D1.RenderTarget(d2DFactory, surface, renderTargetProperties);

            SetRenderTarget(renderTarget);

            device.ImmediateContext.Rasterizer.SetViewport(0, 0, width, height);

            CreationProperties creationProperties = new CreationProperties
            {
                DebugLevel = DebugLevel.Error,
                Options = DeviceContextOptions.EnableMultithreadedOptimizations,
                ThreadingMode = ThreadingMode.MultiThreaded,
            };


            _d2dContext = new DeviceContext(surface, creationProperties);

            _d2DRenderTarget = _d2dContext;

            InvalidateVisual();
        }

        public new void InvalidateVisual()
        {
            Rendering();
        }

        /// <inheritdoc />
        protected override void OnRender(DrawingContext drawingContext)
        {
            drawingContext.DrawImage(_d3D, new Rect(new Size(_d3D.PixelWidth, _d3D.PixelHeight)));
        }

   
        protected abstract void OnRender(SharpDX.Direct2D1.RenderTarget renderTarget);

        private SharpDX.Direct3D9.Texture _renderTarget;
        private SharpDX.Direct2D1.RenderTarget _d2DRenderTarget;
        private DeviceContext _d2dContext;

        private SharpDX.Direct3D11.Device _device;

        private async void Rendering()
        {
            await Task.Run(() =>
            {
                _d2dContext.BeginDraw();

                OnRender(_d2dContext);

                _d2dContext.EndDraw();

                _device.ImmediateContext.Flush();
            });


            _d3D.Lock();

            _d3D.AddDirtyRect(new Int32Rect(0, 0, _d3D.PixelWidth, _d3D.PixelHeight));

            _d3D.Unlock();

            base.InvalidateVisual();
        }

        private void SetRenderTarget(SharpDX.Direct3D11.Texture2D target)
        {
            var format = TranslateFormat(target);
            var handle = GetSharedHandle(target);

            var presentParams = GetPresentParameters();
            var createFlags = SharpDX.Direct3D9.CreateFlags.HardwareVertexProcessing |
                              SharpDX.Direct3D9.CreateFlags.Multithreaded |
                              SharpDX.Direct3D9.CreateFlags.FpuPreserve;

            var d3DContext = new SharpDX.Direct3D9.Direct3DEx();
            var d3DDevice = new SharpDX.Direct3D9.DeviceEx(d3DContext, 0, SharpDX.Direct3D9.DeviceType.Hardware,
                IntPtr.Zero, createFlags,
                presentParams);

            _renderTarget = new SharpDX.Direct3D9.Texture(d3DDevice, target.Description.Width,
                target.Description.Height, 1,
                SharpDX.Direct3D9.Usage.RenderTarget, format, SharpDX.Direct3D9.Pool.Default, ref handle);

            using (var surface = _renderTarget.GetSurfaceLevel(0))
            {
                _d3D.Lock();
                _d3D.SetBackBuffer(D3DResourceType.IDirect3DSurface9, surface.NativePointer);
                _d3D.Unlock();
            }
        }

        private static SharpDX.Direct3D9.PresentParameters GetPresentParameters()
        {
            var presentParams = new SharpDX.Direct3D9.PresentParameters();

            presentParams.Windowed = true;
            presentParams.SwapEffect = SharpDX.Direct3D9.SwapEffect.Discard;
            presentParams.DeviceWindowHandle = NativeMethods.GetDesktopWindow();
            presentParams.PresentationInterval = SharpDX.Direct3D9.PresentInterval.Default;

            return presentParams;
        }

        private IntPtr GetSharedHandle(SharpDX.Direct3D11.Texture2D texture)
        {
            using (var resource = texture.QueryInterface<SharpDX.DXGI.Resource>())
            {
                return resource.SharedHandle;
            }
        }

        private static SharpDX.Direct3D9.Format TranslateFormat(SharpDX.Direct3D11.Texture2D texture)
        {
            switch (texture.Description.Format)
            {
                case SharpDX.DXGI.Format.R10G10B10A2_UNorm:
                    return SharpDX.Direct3D9.Format.A2B10G10R10;
                case SharpDX.DXGI.Format.R16G16B16A16_Float:
                    return SharpDX.Direct3D9.Format.A16B16G16R16F;
                case SharpDX.DXGI.Format.B8G8R8A8_UNorm:
                    return SharpDX.Direct3D9.Format.A8R8G8B8;
                default:
                    return SharpDX.Direct3D9.Format.Unknown;
            }
        }

        private static class NativeMethods
        {
            [DllImport("user32.dll", SetLastError = false)]
            public static extern IntPtr GetDesktopWindow();
        }
    }

```

更多渲染博客请看 [WPF 底层渲染](https://blog.csdn.net/lindexi_gd/column/info/24324 )

特别感谢 

[Direct2D - 随笔分类 - 万仓一黍 - 博客园](http://www.cnblogs.com/grenet/category/507059.html )

[Direct2D - 随笔分类 - 万一 - 博客园](http://www.cnblogs.com/del/category/290814.html )

[Direct2D - 随笔分类 - zdd - 博客园](http://www.cnblogs.com/graphics/category/412802.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



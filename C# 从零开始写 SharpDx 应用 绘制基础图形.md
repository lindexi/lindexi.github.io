# C# 从零开始写 SharpDx 应用 绘制基础图形

本文告诉大家通过 SharpDx 画出简单的 2D 界面

<!--more-->
<!-- CreateTime:2019/10/23 21:16:35 -->


本文属于 [SharpDx 系列](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html) 博客，建议从头开始读

本文分为两步，第一步是初始化，第二步才是画界面

## 初始化

先创建 RenderForm 用来显示界面，在创建的过程需要指定宽度和高度

```csharp
            _renderForm = new RenderForm();
            _renderForm.ClientSize = new Size(Width, Height);

        private const int Width = 1280;

        private const int Height = 720;
```

上面创建的代码大部分参阅了[C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.csdn.net/lindexi_gd/article/details/82114907 )的代码

在 InitializeDeviceResources 函数里面更改一些参数，用于创建资源和初始化

```csharp
            var backBufferDesc =
                new ModeDescription(Width, Height, new Rational(60, 1), Format.R8G8B8A8_UNorm);
            var swapChainDesc = new SwapChainDescription
            {
                BufferCount = 1,
                ModeDescription = backBufferDesc,
                IsWindowed = true,
                OutputHandle = _renderForm.Handle,
                SampleDescription = new SampleDescription(1, 0),
                SwapEffect = SwapEffect.Discard,
                Usage = Usage.RenderTargetOutput
            };

            Device.CreateWithSwapChain(DriverType.Hardware, DeviceCreationFlags.BgraSupport, swapChainDesc,
                out _d3DDevice, out _swapChain);

            _d3DDeviceContext = _d3DDevice.ImmediateContext;

            using (var backBuffer = _swapChain.GetBackBuffer<Texture2D>(0))
            {
                _renderTargetView = new RenderTargetView(_d3DDevice, backBuffer);

                _viewport = new Viewport(0, 0, Width, Height);
                _d3DDeviceContext.Rasterizer.SetViewport(_viewport);
            }

            CreateD2DRender();
```

上面参数和[C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.csdn.net/lindexi_gd/article/details/82114907 )的有一些不相同，在 SwapChainDescription 里面添加了 SwapEffect 参数

在创建交换链的时候，在 Device.CreateWithSwapChain 里面修改了 DeviceCreationFlags 参数

上面内容还是在创建 3D 内容，在 DX 里面是通过一个 3D 的平面画 2D 界面

在 CreateD2DRender 方法里面才是创建 2D 的代码

想要绘制界面需要 `SharpDX.Direct2D1.RenderTarget` 对象，需要先创建工厂然后通过工厂和交换链拿到平面，然后将输出定向到拿到的平面

创建工厂只需要直接创建

```csharp
            var d2dFactory = new SharpDX.Direct2D1.Factory();
```

从交换链拿到平面

```csharp
            Texture2D backBuffer = D3D11.Resource.FromSwapChain<Texture2D>(_swapChain, 0);
            Surface surface = backBuffer.QueryInterface<Surface>();
```

创建 RenderTarget 用于绘图

```csharp
            var d2dRenderTarget = new RenderTarget(d2dFactory, surface,
                new RenderTargetProperties(new PixelFormat(Format.Unknown, AlphaMode.Premultiplied)));
```

这里 CreateD2DRender 方法代码请看下面

```csharp
using DeviceContext = SharpDX.Direct2D1.DeviceContext;
using Factory = SharpDX.Direct2D1.Factory;

        private void CreateD2DRender()
        {
            var d2dFactory = new SharpDX.Direct2D1.Factory();
            Texture2D backBuffer = D3D11.Resource.FromSwapChain<Texture2D>(_swapChain, 0);
            Surface surface = backBuffer.QueryInterface<Surface>();
            var d2dRenderTarget = new RenderTarget(d2dFactory, surface,
                new RenderTargetProperties(new PixelFormat(Format.Unknown, AlphaMode.Premultiplied)));
            _d2dFactory = d2dFactory;
            _d2dRenderTarget = d2dRenderTarget;
        }

        private Factory _d2dFactory;
        private RenderTarget _d2dRenderTarget;
```

现在拿到了 `_d2dRenderTarget` 就可以用来画 2D 的界面了，开启绘制循环之后进行画界面

```csharp
        public void Run()
        {
            RenderLoop.Run(_renderForm, RenderCallback);
        }

        private void RenderCallback()
        {
            Draw();
        }

        private void Draw()
        {
        	// 在这里绘制
        }
```

下面将会告诉大家如何在 Draw 方法里面绘制界面

## 画界面

在 Draw 方法里面，使用下面方式画界面

```csharp
        private void Draw()
        {
            _d2dRenderTarget.BeginDraw();
           
            // 实际画的代码

            _d2dRenderTarget.EndDraw();


            _swapChain.Present(1, PresentFlags.None);
        }
```

先调用 BeginDraw 方法开启绘制，在调用 EndDraw 方法将所有绘制指令压缩处理，大部分都是直接传送到显卡渲染

然后调用交换链 `_swapChain` 将后台缓存和前台显示交换，这样就可以做到刷新界面

具体画的内容可以分为基础图形和 3D 绘制

在所有开始绘制之前都需要调用 BeginDraw 方法，在绘制完成之后调用 EndDraw 方法将绘制的命令处理，然后发送到显卡

### 画线

画线条需要传入两个点，用两个点画出一条线条，还有线条的笔刷。可选的是线条的宽度，和样式

下面代码是作为添加所有参数的例子

```csharp
            _d2dRenderTarget.BeginDraw();

            var brush = new SolidColorBrush(_d2dRenderTarget, ColorToRaw4(Color.Bisque));
            var styleProperties = new StrokeStyleProperties()
            {
                StartCap = CapStyle.Square,
                EndCap = CapStyle.Round
            };
            var strokeStyle = new StrokeStyle(_d2dFactory, styleProperties);

            using (strokeStyle)
            using (brush)
            {
                _d2dRenderTarget.DrawLine(new RawVector2(10, 100), new RawVector2(100, 100),
                    brush, strokeWidth: 5, strokeStyle);
            }

            _d2dRenderTarget.EndDraw();
```

运行代码可以看到下面图片

<!-- ![](image/C# 从零开始写 SharpDx 应用 绘制基础图形/C# 从零开始写 SharpDx 应用 绘制基础图形0.png) -->

![](http://image.acmx.xyz/lindexi%2F2019616202452970)

这段代码写在 Draw 函数里面，在 SharpDx 里面创建的资源，例如笔画和样式等，都需要做手动的释放，这部分的写法和 WPF 不相同，需要自己关注资源的创建和释放，但是这样做才能做到更改的性能

在 StrokeStyleProperties 里面有很多有趣的参数，请大家自己玩一下

在代码里面用到 ColorToRaw4 方法，因为在 SharpDx 里面的颜色使用的范围是 0-1 而不是 System.Drawing.Color 使用 255 范围

```csharp
        RawColor4 ColorToRaw4(Color color)
        {
            const float n = 255f;
            return new RawColor4(color.R / n, color.G / n, color.B / n, color.A / n);
        }
```

通过 ColorToRaw4 方法可以转换颜色

### 矩形

通过 DrawRectangle 方法可以画出矩形，在矩形里面需要传入 RawRectangleF 和颜色，可选线条宽度和样式和线条相同

```csharp
            var brush = new SolidColorBrush(_d2dRenderTarget, ColorToRaw4(Color.Bisque));
            var rect = new RawRectangleF(10, 10, 100, 100);
           
            using (brush)
            {
                _d2dRenderTarget.DrawRectangle(rect, brush);
            }
```

注意 RawRectangleF 的构造函数传入的是左上右下而不是左上角的点和宽度高度

```csharp
 var rect = new RawRectangleF(left: 10, top: 10, right: 100, bottom: 100);
```

在没有指定线条宽度是会使用 `_d2dRenderTarget` 的默认线条宽度，通过下面代码可以设置默认线条宽度

```csharp
            _d2dRenderTarget.StrokeWidth = 10;
```

圆角矩形可以使用 DrawRoundedRectangle 方法

```csharp
                var roundedRectangle = new RoundedRectangle();
                roundedRectangle.Rect = rect;
                roundedRectangle.RadiusX = 10;
                roundedRectangle.RadiusY = 10;
                _d2dRenderTarget.DrawRoundedRectangle(roundedRectangle, brush);
```

这里的 rect 和 brush 都是上面的代码

填充矩形使用 FillRectangle 方法，这个方法只需要传入矩形和笔刷，稍微更改上面的代码

```csharp
                _d2dRenderTarget.FillRectangle(rect, brush);
```

运行代码你可以看到一个填充的矩形

填充的圆角矩形使用 FillRoundedRectangle 方法，这个方法也不需要传入线条宽度等

```csharp
                _d2dRenderTarget.FillRoundedRectangle(roundedRectangle, brush);
```

运行上面代码，可以看到填充的圆角矩形

### 椭圆

画椭圆使用 DrawEllipse 方法，传入椭圆和线条颜色，可选线条宽度和样式

```csharp
            var brush = new SolidColorBrush(_d2dRenderTarget, ColorToRaw4(Color.Bisque));

            var ellipse = new Ellipse(new RawVector2(100, 100), 10, 10);

            using (brush)
            {
                _d2dRenderTarget.DrawEllipse(ellipse, brush);
            }
```

运行上面代码可以看到下图

<!-- ![](image/C# 从零开始写 SharpDx 应用 绘制基础图形/C# 从零开始写 SharpDx 应用 绘制基础图形1.png) -->

![](http://image.acmx.xyz/lindexi%2F2019616204315473)

创建椭圆时传入的是圆心和两个方向的大小

填充椭圆使用 FillEllipse 方法，传入的是笔刷，不需要传入线条宽度等

```csharp
            var brush = new SolidColorBrush(_d2dRenderTarget, ColorToRaw4(Color.Bisque));

            var ellipse = new Ellipse(new RawVector2(100, 100), 10, 10);

            using (brush)
            {
                _d2dRenderTarget.FillEllipse(ellipse, brush);
            }
```

使用上面代码可以填充椭圆

## 几何

复杂的几何可以使用 Geometry 绘制

使用 DrawGeometry 方法传入 Geometry 和颜色，可选线条相关设置

```csharp
            var brush = new SolidColorBrush(_d2dRenderTarget, ColorToRaw4(Color.Bisque));

            var rect = new RawRectangleF(left: 10, top: 10, right: 100, bottom: 100);
            Geometry geometry = new RectangleGeometry(_d2dFactory, rect);

            using(geometry)
            using (brush)
            {
                _d2dRenderTarget.DrawGeometry(geometry, brush);
            }
```

这里的 Geometry 可选的很多，最支持定制的是 PathGeometry 方法

如使用很多代码画出线条

```csharp
            var geometry = new PathGeometry(_d2dFactory);
            var geometrySink = geometry.Open();
            geometrySink.BeginFigure(new RawVector2(10,100),FigureBegin.Filled );
            geometrySink.AddLine(new RawVector2(100,100));
            geometrySink.EndFigure(FigureEnd.Closed);
            geometrySink.Close();
```

在 PathGeometry 使用 Open 方法返回 GeometrySink 可以支持很多绘制，包括组合多个几何

### 文字

绘制文字需要 `SharpDX.DirectWrite.Factory` 需要先创建才能使用，注意工厂需要只创建一次

```csharp
            var factory = new SharpDX.DirectWrite.Factory();
```

创建工厂可以用来实例文本格式

```csharp
var textFormat = new TextFormat(factory, "宋体", 20);
```

在 TextFormat 构造函数可以传入很多参数，用于绘制

绘制文本需要使用 DrawText 方法，在这个方法传入需要绘制的字符串和文本格式，和绘制的范围和颜色

```csharp
            var brush = new SolidColorBrush(_d2dRenderTarget, ColorToRaw4(Color.Bisque));

            var factory = new SharpDX.DirectWrite.Factory();
            var textFormat = new TextFormat(factory, "宋体", 20);

            var rect = new RawRectangleF(left: 10, top: 10, right: 100, bottom: 100);

            using(textFormat)
            using (brush)
            {
                _d2dRenderTarget.DrawText("林德熙是逗比", textFormat, rect, brush);
            }
```

对于 factory 在实际项目请写在初始化的方法，而不是每次进入绘制方法的时候都创建，这个代码将会内存泄露

在画文本需要用到很多参数，用于自己定制，请小伙伴自己玩一下

有了基础的画界面就可以做出好看的界面，如何根据这些简单的方法画出好看的界面请看 [WPF 源代码 从零开始写一个 UI 框架](https://blog.lindexi.com/post/WPF-%E6%BA%90%E4%BB%A3%E7%A0%81-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99%E4%B8%80%E4%B8%AA-UI-%E6%A1%86%E6%9E%B6.html )

更多请看 [SharpDx 系列](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html)

使用 SharpDx 绘制很底层，但是绘制性能超级高

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

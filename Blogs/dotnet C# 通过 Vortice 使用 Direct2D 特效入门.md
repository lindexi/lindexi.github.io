本文将告诉大家如何通过 Vortice 使用 D2D 的特效

<!--more-->


<!-- CreateTime:2023/5/22 8:51:31 -->

<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

本文属于 DirectX 系列博客，更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

上一篇： [DirectX 使用 Vortice 从零开始控制台创建 Direct2D1 窗口修改颜色](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html )

在上一篇博客里面，咱创建了一个 Win32 空窗口，接着给他挂上了 DirectX 交换链。使用以下代码从交换链里面拿到了 DXGI 平面，拿到的的 DXGI 平面即可被绘制 2D 内容在上面，从而将内容绘制输出到窗口上

```csharp
        DXGI.IDXGISwapChain1 swapChain = ... // 忽略交换链之前的代码

        D3D11.ID3D11Texture2D backBufferTexture = swapChain.GetBuffer<D3D11.ID3D11Texture2D>(0);

        // 获取到 dxgi 的平面，这个屏幕就约等于窗口渲染内容
        DXGI.IDXGISurface dxgiSurface = backBufferTexture.QueryInterface<DXGI.IDXGISurface>();
```

接下来咱将创建 D2D 设备，再通过 D2D 设备，从而拿到 ID2D1DeviceContext 对象，设置 DXGI 平面作为 ID2D1DeviceContext 输出，如此即可使用继承自 ID2D1RenderTarget 的 ID2D1DeviceContext 进行绘制界面。此步骤更详细的内容，请参阅 [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html )

```csharp
        DXGI.IDXGIDevice dxgiDevice = d3D11Device.QueryInterface<DXGI.IDXGIDevice>();
        ID2D1Device d2dDevice = d2DFactory.CreateDevice(dxgiDevice);
        ID2D1DeviceContext d2dDeviceContext = d2dDevice.CreateDeviceContext();

        // 设置 DXGI 平面作为 ID2D1DeviceContext 输出
        ID2D1Bitmap1 d2dBitmap = d2dDeviceContext.CreateBitmapFromDxgiSurface(dxgiSurface);
        d2dDeviceContext.Target = d2dBitmap;
```

使用 Direct2D 特效之前，需要先了解一些概念。在 Direct2D 里面，可使用 ID2D1Effect 特效，特效的常用方法就是先使用 SetInput 方法将某个 ID2D1Bitmap 设置为输入源，再通过 SetValue 设置一些参数。最后将特效放入到 ID2D1DeviceContext 的 DrawImage 方法绘制出来

在 Direct2D 里面有许多内建特效，作为入门的博客，咱随便选一个简单的 GaussianBlur 特效作为例子。先通过 ID2D1DeviceContext 创建出特效，代码如下

```csharp
                var gaussianBlurEffect = d2dDeviceContext.CreateEffect(EffectGuids.GaussianBlur);
                using ID2D1Effect d2dEffect = new ID2D1Effect(gaussianBlurEffect);
```

创建出来的特效现在是缺少输入源的，接下来咱随便写一点代码创建一个 ID2D1Bitmap 作为输入源。为了方便编写例子，我这里采用的是创建放在内存里的 IWICBitmap 作为画布，通过自己绘制的内容作为 ID2D1Bitmap 输出，方法如下

```csharp
        ID2D1Bitmap CreateBitmap()
        {
            using var wicImagingFactory = new IWICImagingFactory();
            using IWICBitmap wicBitmap =
                wicImagingFactory.CreateBitmap(1000, 1000, Win32.Graphics.Imaging.Apis.GUID_WICPixelFormat32bppPBGRA);
            var renderTargetProperties = new D2D.RenderTargetProperties(Vortice.DCommon.PixelFormat.Premultiplied);
            using D2D.ID2D1RenderTarget wicBitmapRenderTarget =
                d2DFactory.CreateWicBitmapRenderTarget(wicBitmap, renderTargetProperties);
            wicBitmapRenderTarget.BeginDraw();
            using var brush = wicBitmapRenderTarget.CreateSolidColorBrush(color);
            wicBitmapRenderTarget.FillEllipse(new Ellipse(new System.Numerics.Vector2(200, 200), 100, 100), brush);
            wicBitmapRenderTarget.EndDraw();

            ID2D1Bitmap1 intputBitmap = renderTarget.CreateBitmapFromWicBitmap(wicBitmap);
            return intputBitmap;
        }
```

以上代码不是本文的重点，大家也可以采用加载本地图片等方式获取到 ID2D1Bitmap 对象。如何加载本机图片，请参阅 [WPF 对接 Vortice 在 Direct2D 绘制从 WIC 加载的图片](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E5%9C%A8-Direct2D-%E7%BB%98%E5%88%B6%E4%BB%8E-WIC-%E5%8A%A0%E8%BD%BD%E7%9A%84%E5%9B%BE%E7%89%87.html )

获取到 ID2D1Bitmap 对象，即可调用 SetInput 方法设置输入源，代码如下

```csharp
                using ID2D1Bitmap intputBitmap = CreateBitmap();

                d2dEffect.SetInput(0, intputBitmap, new RawBool(true));
```

接着使用 SetValue 方法设置特效的一些参数，特效的参数都是灵活的。此 SetValue 方法的第一个参数表示的是将要设置特效的哪个参数，第二个参数才是特效参数的值，例如以下代码，设置模糊度

```csharp
                const int D2D1_GAUSSIANBLUR_PROP_STANDARD_DEVIATION = 0;
                d2dEffect.SetValue(D2D1_GAUSSIANBLUR_PROP_STANDARD_DEVIATION, 2.5f);
```

完成特效的设置之后，即可调用 DrawImage 方法将特效结果绘制出来，代码如下

```csharp
                renderTarget.DrawImage(d2dEffect);
```

以上就是特效的入门玩法，可以通过高性能的 Direct2D 特效，绘制出绚丽的界面

下面是特效使用部分的代码

```csharp
        // 创建 D2D 设备，通过设置 ID2D1DeviceContext 的 Target 输出为 dxgiSurface 从而让 ID2D1DeviceContext 渲染内容渲染到窗口上
        // 如 https://learn.microsoft.com/en-us/windows/win32/direct2d/images/devicecontextdiagram.png 图
        // 获取 DXGI 设备，用来创建 D2D 设备
        DXGI.IDXGIDevice dxgiDevice = d3D11Device.QueryInterface<DXGI.IDXGIDevice>();
        ID2D1Device d2dDevice = d2DFactory.CreateDevice(dxgiDevice);
        ID2D1DeviceContext d2dDeviceContext = d2dDevice.CreateDeviceContext();

        ID2D1Bitmap1 d2dBitmap = d2dDeviceContext.CreateBitmapFromDxgiSurface(dxgiSurface);
        d2dDeviceContext.Target = d2dBitmap;

        var renderTarget = d2dDeviceContext;

        var stopwatch = Stopwatch.StartNew();
        var count = 0;

        // 随意创建颜色
        var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255),
            (byte) Random.Shared.Next(255));

        Task.Factory.StartNew(() =>
        {
            while (true)
            {
                // 开始绘制逻辑
                renderTarget.BeginDraw();

                // 清空画布
                renderTarget.Clear(new Color4(0xFF,0xFF,0xFF));

                // 随便创建一张图片
                using var inputBitmap = CreateBitmap();

                var gaussianBlurEffect = d2dDeviceContext.CreateEffect(EffectGuids.GaussianBlur);
                using ID2D1Effect d2dEffect = new ID2D1Effect(gaussianBlurEffect);

                d2dEffect.SetInput(0, inputBitmap, new RawBool(true));
                const int D2D1_GAUSSIANBLUR_PROP_STANDARD_DEVIATION = 0;
                d2dEffect.SetValue(D2D1_GAUSSIANBLUR_PROP_STANDARD_DEVIATION, count / 60f * 3f);

                renderTarget.DrawImage(d2dEffect);

                renderTarget.EndDraw();

                swapChain.Present(1, DXGI.PresentFlags.None);
                // 等待刷新
                d3D11DeviceContext.Flush();

                // 统计刷新率
                count++;
                if (stopwatch.Elapsed >= TimeSpan.FromSeconds(1))
                {
                    Console.WriteLine($"FPS: {count / stopwatch.Elapsed.TotalSeconds}");
                    stopwatch.Restart();
                    count = 0;
                }
            }
        }, TaskCreationOptions.LongRunning);

        ID2D1Bitmap CreateBitmap()
        {
            using var wicImagingFactory = new IWICImagingFactory();
            using IWICBitmap wicBitmap =
                wicImagingFactory.CreateBitmap(1000, 1000, Win32.Graphics.Imaging.Apis.GUID_WICPixelFormat32bppPBGRA);
            var renderTargetProperties = new D2D.RenderTargetProperties(Vortice.DCommon.PixelFormat.Premultiplied);
            using D2D.ID2D1RenderTarget wicBitmapRenderTarget =
                d2DFactory.CreateWicBitmapRenderTarget(wicBitmap, renderTargetProperties);
            wicBitmapRenderTarget.BeginDraw();
            using var brush = wicBitmapRenderTarget.CreateSolidColorBrush(color);
            wicBitmapRenderTarget.FillEllipse(new Ellipse(new System.Numerics.Vector2(200, 200), 100, 100), brush);
            wicBitmapRenderTarget.EndDraw();

            ID2D1Bitmap1 inputBitmap = renderTarget.CreateBitmapFromWicBitmap(wicBitmap);
            return inputBitmap;
        }
```

以上代码将可以动态绘制一个模糊度不断变化的圆，代码省略和没有写的部分，我放在了 [github](https://github.com/lindexi/lindexi_gd/tree/615c235ce34b8c38abe1e99e65a5e34ddc9addb0/VorticeD2DEffect1) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/615c235ce34b8c38abe1e99e65a5e34ddc9addb0/VorticeD2DEffect1) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 615c235ce34b8c38abe1e99e65a5e34ddc9addb0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 615c235ce34b8c38abe1e99e65a5e34ddc9addb0
```

获取代码之后，进入 VorticeD2DEffect1 文件夹

有伙伴好奇为什么我最近写的是通过 Vortice 调用 DirectX 的博客，而不是通过 SharpDx 或 Silk.NET 调用 DirectX 的博客。这不是我收了 Vortice 的钱或者是和 Vortice 有什么 py 交易哈。其原因是 SharpDx 不维护了，作为 SharpDx 的接任者 Vortice 的行为和 API 都会靠近 SharpDx 许多，我编写起来比较顺手。而 Silk.NET 是对 DirectX 的底层封装，由于是直接底层封装，导致使用 Silk.NET 比较繁琐。尽管使用 Silk.NET 的性能从理论分析上能够比 Vortice 和 SharpDx 更好，但从定量上说，其实好不了多少。我所遇到的几乎所有性能问题，基本都卡在渲染上，而不是调用上，调用上的损耗基本可以忽略。那 Silk.NET 是不是就无用武之地？其实不然，在一些情况下，机器的性能不够业务的需求情况下，能省多少就应该省多少。而且在熟悉整个过程之后，即使将 Vortice 换成 Silk.NET 也只不过是一个体力活而已，将各个 API 进行替换即可。而且有趣的是，可以混合着 Vortice 和 Silk.NET 一起用，只有某些模块才使用 Silk.NET 编写

我创建了专门聊 Vortice 的 QQ 群： 622808968 欢迎加入交流技术

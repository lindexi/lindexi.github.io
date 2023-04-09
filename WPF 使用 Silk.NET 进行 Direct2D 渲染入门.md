# WPF 使用 Silk.NET 进行 Direct2D 渲染入门

在上一篇博客的基础上，使用 dotnet 基金会新开源的 Silk.NET 库，让 Silk.NET 创建的 DX 设备和 WPF 对接渲染。接下来本文将告诉大家如何使用 Silk.NET 提供的 Direct2D 底层封装，在 WPF 上绘制出界面

<!--more-->

<!-- 标签：WPF,DirectX,渲染 -->
<!-- 博客 -->
<!-- 发布 -->

接着上一篇博客 [WPF 使用 Silk.NET 进行 DirectX 渲染入门](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Silk.NET-%E8%BF%9B%E8%A1%8C-DirectX-%E6%B8%B2%E6%9F%93%E5%85%A5%E9%97%A8.html ) 的内容，在上一篇博客编写时，官方 Silk.NET 仓库还没完成对 Direct2D 的支持。因此上一篇博客采用的是 SharpDx 进行 Direct2D 的对接。本篇博客将换掉 SharpDx 库，完全采用 Silk.NET 完成 Direct2D 的对接，在 WPF 上绘制出界面

采用 Silk.NET 库编写 DirectX 逻辑，将会发现由于 Silk.NET 库是 DirectX 的底层封装库，写时需要了解的细节非常多，而且代码也写起来不够优雅。基本上需要完全开启不安全代码写起来才方便一些，但即使如此，编写起来的代码逻辑相比于 SharpDx 库来说还是复杂许多。采用 Silk.NET 库可以提供的一个优势在于对 DirectX 完全底层的封装，可以实现高可控，通过高可控可以实现更高的性能

本文的所有代码都可以在本文末尾找到下载方式

## 初始化项目

先在[上一篇博客](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Silk.NET-%E8%BF%9B%E8%A1%8C-DirectX-%E6%B8%B2%E6%9F%93%E5%85%A5%E9%97%A8.html )的代码基础上，卸掉 SharpDx 相关库，换成 Silk.NET.Direct2D 库。修改之后的 csproj 项目文件的内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <AllowUnsafeBlocks>True</AllowUnsafeBlocks>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Silk.NET.Direct2D" Version="2.17.0" />
    <PackageReference Include="Silk.NET.Direct3D11" Version="2.17.0" />
    <PackageReference Include="Silk.NET.Direct3D9" Version="2.17.0" />
    <PackageReference Include="Silk.NET.DXGI" Version="2.17.0" />
  </ItemGroup>

</Project>
```

卸掉 SharpDx 库之后，同步在代码里面删除所有和 SharpDx 相关的代码，接下来将换成使用 Silk.NET 提供的 Direct2D 封装进行对接

## 创建工厂

按照 DirectX 对接的惯例，在对接 Direct2D 时，需要先创建出 Direct2D 的工厂。为了防止命名冲突，先引用命名空间，如以下代码

```csharp
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Interop;
using System.Windows.Media;

using Silk.NET.Core.Native;

using D3D11 = Silk.NET.Direct3D11;
using D3D9 = Silk.NET.Direct3D9;
using DXGI = Silk.NET.DXGI;
using D2D = Silk.NET.Direct2D;
using Silk.NET.Direct2D;
using Silk.NET.Maths;
using Silk.NET.Direct3D11;
```

使用 Silk.NET 创建 Direct2D 的工厂的代码如下

```csharp
        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
        	...// 忽略其他代码

            D2D.ID2D1Factory* pD2D1Factory = (D2D.ID2D1Factory*) IntPtr.Zero;
            var d2D = D2D.D2D.GetApi();
            Guid guid = D2D.ID2D1Factory.Guid;
            hr = d2D.D2D1CreateFactory(D2D.FactoryType.SingleThreaded, ref guid, new D2D.FactoryOptions(D2D.DebugLevel.Error),
                ((void**) &pD2D1Factory));
            SilkMarshal.ThrowHResult(hr);
            _pD2D1Factory = pD2D1Factory;

        	...// 忽略其他代码
        }

        private D2D.ID2D1Factory* _pD2D1Factory;
```

在开始做实际创建之前，需要使用 `D2D.GetApi` 获取到封装的 D2D 辅助类。由于 Silk.NET 的 API 还不稳定，本文这里不对 GetApi 方法进行更新，保持原先版本的逻辑

除了以上方式创建 ID2D1Factory 工厂之外，还可以通过从 ID3D11Device 里面获取到的 IDXGIDevice 进行创建 ID2D1Device 设备，然后从 ID2D1Device 设备里面获取到 ID2D1Factory 工厂。这个路线稍微绕一点

通过 [上一篇博客](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Silk.NET-%E8%BF%9B%E8%A1%8C-DirectX-%E6%B8%B2%E6%9F%93%E5%85%A5%E9%97%A8.html ) 获取到的 `D3D11.ID3D11Device* pD3D11Device;` 转换为 `IDXGIDevice` 设备，代码如下

```csharp
            // 通过 DXGI 创建
            IDXGIDevice* pDXGIDevice = pD3D11Device->QueryInterface<IDXGIDevice>().Handle;
```

接着通过 `IDXGIDevice* pDXGIDevice` 创建出 D2D 设备，代码如下

```csharp
            ID2D1Device* pD2D1Device;
            var creationProperties = new CreationProperties(ThreadingMode.SingleThreaded, DebugLevel.Error, DeviceContextOptions.None);
            d2D.D2D1CreateDevice(pDXGIDevice, creationProperties, &pD2D1Device);
```

拿到 ID2D1Device 设备即可调用 GetFactory 方法获取到工厂，代码如下

```csharp
            pD2D1Device->GetFactory(&pD2D1Factory);

            _pD2D1Factory = pD2D1Factory;
```

## 创建 ID2D1RenderTarget 对象

根据 D2D 的玩法，通用的使用方法是创建出 ID2D1RenderTarget 作为画布的角色，接着即可在 ID2D1RenderTarget 上绘制各个东西

在本文这里，创建 ID2D1RenderTarget 时，需要关注的是创建的 ID2D1RenderTarget 是将输出到哪里。在 [上一篇博客](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Silk.NET-%E8%BF%9B%E8%A1%8C-DirectX-%E6%B8%B2%E6%9F%93%E5%85%A5%E9%97%A8.html ) 创建出了一个 `D3D11.ID3D11Texture2D* pD3D11Texture2D` 纹理，接着还将这个纹理共享给了 `D3D9.IDirect3DTexture9* pDirect3DTexture9` 纹理，从而对接到 WPF 的 D3DImage 里面

如此即可知道，咱需要的是让 ID2D1RenderTarget 输出到 [上一篇博客](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Silk.NET-%E8%BF%9B%E8%A1%8C-DirectX-%E6%B8%B2%E6%9F%93%E5%85%A5%E9%97%A8.html ) 创建出了一个 `D3D11.ID3D11Texture2D* pD3D11Texture2D` 纹理上。这样就可以通过纹理的共享，将画面输出到 WPF 的 D3DImage 里面，从而在界面上显示

在 D3D11 的 ID3D11Texture2D 上通过 ID2D1RenderTarget 绘制内容，需要将 ID2D1RenderTarget 对象创建出来。先将 ID3D11Texture2D 转换为 IDXGISurface 用于进行和 D2D 对接，代码如下

```csharp
            DXGI.IDXGISurface* pDXGISurface;
            var dxgiSurfaceGuid = DXGI.IDXGISurface.Guid;
            renderTarget->QueryInterface(ref dxgiSurfaceGuid, (void**) &pDXGISurface);
            _pDXGISurface = pDXGISurface;
```

接着使用上文创建出的 D2D 工厂，调用 CreateDxgiSurfaceRenderTarget 创建出 ID2D1RenderTarget 对象，代码如下

```csharp
            var renderTargetProperties =
            new D2D.RenderTargetProperties(pixelFormat: new D2D.PixelFormat(DXGI.Format.FormatUnknown,
                D2D.AlphaMode.Premultiplied), type: D2D.RenderTargetType.Hardware, dpiX: 96, dpiY: 96, usage: D2D.RenderTargetUsage.None, minLevel: D2D.FeatureLevel.LevelDefault);

            D2D.ID2D1RenderTarget* pD2D1RenderTarget;
            hr = pD2D1Factory->CreateDxgiSurfaceRenderTarget(pDXGISurface, renderTargetProperties, &pD2D1RenderTarget);
            SilkMarshal.ThrowHResult(hr);

            _pD2D1RenderTarget = pD2D1RenderTarget;
```

如此即可拿到 ID2D1RenderTarget 对象，可以将其当成画布，在上面绘制内容

## 绘制界面

拿到 ID2D1RenderTarget 对象之后，可以在 WPF 的 `CompositionTarget.Rendering` 事件里面开始绘制逻辑

值得一提的是，一旦加等了 WPF 的 `CompositionTarget.Rendering` 事件，无论是否执行任何逻辑，都会降低 WPF 的性能。好在只是降低一点点。多次加等无此问题，多次加等的性能损耗取决于具体业务代码执行时间，不会影响到 WPF 基础框架层的性能。这是因为在 WPF 框架里面，判断了 `CompositionTarget.Rendering` 事件的逻辑，一旦没有任何业务代码加等，就可以少跑很多逻辑从而提升性能

```csharp
            CompositionTarget.Rendering += CompositionTarget_Rendering;
```

在 `CompositionTarget_Rendering` 里面需要执行两个事情，一个就是通过 `D2D.ID2D1RenderTarget* _pD2D1RenderTarget` 对象绘制 D2D 部分的内容，另一个就是调用 D3DImage 的刷新

先编写核心的 ID2D1RenderTarget 绘制逻辑，这部分大家可以在具体绘制逻辑发挥想象力。在开始绘制之前，需要调用 BeginDraw 通知绘制的开始。且需要确保成对调用 EndDraw 方法。在 BeginDraw 和 EndDraw 中间即可插入具体绘制逻辑，如本文将绘制出许多个矩形

完成之后，此时还没有真正在 GPU 工作，需要调用 ID3D11DeviceContext 的 Flush 将绘制的命令输出到 GPU 里工作。其实准确来说是输出到具体的渲染绘制组件里面，不一定是 GPU 进行处理

值得一提的是，调用 ID3D11DeviceContext 的 Flush 不是在所有的 Direct2D 绘制里都需要，仅仅只是在本例子代码里面的情况下，需要发送更新而已。而且本例子代码里面也没有真的等待绘制完成，在本例子里面的执行的绘制矩形等逻辑，在方法调用完成时，只是将绘制命令发送到 DirectX 的内部命令缓冲区里面，调用 Flush 命令可以将命令发送到真正处理绘制的组件里面，约等于发送到 GPU 执行

大概的代码如下

```csharp
        private void CompositionTarget_Rendering(object? sender, EventArgs e)
        {
            _pD2D1RenderTarget->BeginDraw();

            ...// 绘制业务逻辑

            var hr = _pD2D1RenderTarget->EndDraw((ulong*) IntPtr.Zero, (ulong*) IntPtr.Zero);
            SilkMarshal.ThrowHResult(hr);

            _pD3D11DeviceContext->Flush();

            D3DImage.Lock();

            D3DImage.AddDirtyRect(new Int32Rect(0, 0, D3DImage.PixelWidth, D3DImage.PixelHeight));

            D3DImage.Unlock();
        }
```

以上的代码就是先调用 BeginDraw 方法，执行具体的绘制业务逻辑。从推荐上来说，应该是在 `CompositionTarget_Rendering` 方法进来之前就应该完成绘制逻辑了，而且绘制的逻辑可以放在其他线程执行，这样可以更加充分利用 GPU 和 CPU 的资源

完成绘制逻辑，调用 EndDraw 方法，调用时可以传入实际绘制的需要更新的范围，从而提升一些性能。默认使用 `(ulong*) IntPtr.Zero` 表示全部根据。如以下代码设置更新给定的范围

```csharp
            var rect = new Box2D<float>(0,0,1000,1000);
            var arg1 = (ulong*) &rect;
            var hr = _pD2D1RenderTarget->EndDraw(arg1, (ulong*) IntPtr.Zero);
```

接着调用 Flush 将 DirectX 的绘制命令发送到 GPU 执行

接着通过 D3DImage 的 AddDirtyRect 方法，告诉 WPF 层更新界面内容。同样的可以给定更新需要刷新的范围，从而提升一点性能

本文的绘制业务逻辑是绘制出矩形，绘制矩形需要创建矩形绘制的画刷和线条类型，绘制的代码如下

```csharp
            ID2D1RenderTarget* renderTarget = _pD2D1RenderTarget;

            // 清空画布内容，不在上次的基础上继续绘制
            renderTarget->Clear(null);

            // 创建画刷
            var d3Dcolorvalue = new DXGI.D3Dcolorvalue(r: Random.Shared.NextSingle(), g: Random.Shared.NextSingle(), b: 0, a: 1);
            var brushProperties = new BrushProperties()
            {
                Opacity = 1f,
                Transform = Matrix3X2<float>.Identity,
            };
            ID2D1SolidColorBrush* pSolidColorBrush;
            var hr = renderTarget->CreateSolidColorBrush(d3Dcolorvalue, brushProperties, &pSolidColorBrush);
            SilkMarshal.ThrowHResult(hr);
            ID2D1Brush* pBrush = (ID2D1Brush*) pSolidColorBrush;

            // 创建线条类型
            ID2D1StrokeStyle* pStrokeStyle;
            var strokeStyleProperties = new StrokeStyleProperties()
            {
                StartCap = CapStyle.Square,
                EndCap = CapStyle.Square,
                LineJoin = LineJoin.Bevel
            };

            var pDashes = (float*) IntPtr.Zero;
            hr = _pD2D1Factory->CreateStrokeStyle(strokeStyleProperties, pDashes, 0, &pStrokeStyle);
            SilkMarshal.ThrowHResult(hr);

            // 绘制矩形
            renderTarget->DrawRectangle(pRect, pBrush, 1, pStrokeStyle);

            // 释放引用计数。这里仅仅只是释放引用计数而已，不代表资源被释放
            pSolidColorBrush->Release();
            pStrokeStyle->Release();
```

如此即可完成绘制矩形逻辑

我还在绘制的方法上添加了用来粗略计算 FPS 的逗比代码

```csharp
            _stopwatch ??= Stopwatch.StartNew();
            _renderCount++;
            if (_stopwatch.Elapsed.TotalSeconds > 1)
            {
            	// 在标题显示
                Title = $"FPS: {_renderCount / _stopwatch.Elapsed.TotalSeconds}";
                _renderCount = 0;
                _stopwatch.Restart();
            }

        private int _renderCount = 0;
        private Stopwatch _stopwatch;
```

如果 Direct2D 绘制的具体逻辑足够简单，如就画几千个矩形，那基本上在更差的电脑上的刷新率也是能够达到显示器刷新速度的。但是随着具体项目的业务逻辑的复杂提升，在实际项目里面的刷新率也许会达不到显示器的刷新速度，但这时候大部分的卡性能都不是在渲染上，而是具体的业务逻辑

另外，本文的例子只是一个入门级的使用 Silk.NET 进行 Direct2D 渲染的例子而已，而不是最佳实践，更不是性能提升教程

通过本文的例子可以看到，使用 Silk.NET 可以完成 Direct2D 的绘制功能，而且 Silk.NET 是 DirectX 的底层封装，许多代码写起来不够优雅。但就是因为 Silk.NET 是 DirectX 的底层封装，在一些性能敏感的地方，通过高可控可以减少调用损耗，从而提升一点性能

## 获取代码

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/76fc7b2640b92159364d4f0f83a1fb37b32dcd7a/RawluharkewalQeaninanel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/76fc7b2640b92159364d4f0f83a1fb37b32dcd7a/RawluharkewalQeaninanel) 欢迎访问

可以通过如下方式获取源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 76fc7b2640b92159364d4f0f83a1fb37b32dcd7a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 76fc7b2640b92159364d4f0f83a1fb37b32dcd7a
```

获取代码之后，进入 RawluharkewalQeaninanel 文件夹

更多渲染相关，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

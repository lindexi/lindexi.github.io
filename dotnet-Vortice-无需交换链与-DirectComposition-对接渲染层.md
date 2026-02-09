
# dotnet Vortice 无需交换链与 DirectComposition 对接渲染层

在传统的写法里面，都是面向于 DXGI 交换链实现界面渲染。在 DirectComposition 里面可以通过 IDCompositionDevice 的 Commit 和 WaitForCommitCompletion 方法配置将窗口内容提交到 DWM（DWM Desktop Window Manager）进行渲染，整个过程无需交换链参与 

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

在 DirectComposition 里面提供了 Commit 机制，一次 Commit 的所有内容都能在相同的一帧在屏幕显示出来，如此可以非常方便地完成渲染对齐任务

通过 WaitForCommitCompletion 方法可以等待 Commit 内容完成渲染，此方法作用相当于等待交换链写法的等待垂直同步实现

在 [上一篇博客](https://blog.lindexi.com/post/Vortice-%E4%BD%BF%E7%94%A8-DirectComposition-%E6%98%BE%E7%A4%BA%E9%80%8F%E6%98%8E%E7%AA%97%E5%8F%A3.html ) 中，采用了传统的 DXGI 交换链与 DirectComposition 对接
<!-- [Vortice 使用 DirectComposition 显示透明窗口 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19541356 ) -->

在本文这里将去掉交换链，可以很大简化对接渲染的逻辑。采用 DXGI 交换链对接的方式，可以比较方便对接原有的程序，且可以实现更高帧率的控制。而采用 DirectComposition 的 Commit 写法，可以更好利用 DirectComposition 机制，实现多表面合成以及更加实时的合成器动画

本文将给出最简实现对接的代码逻辑，其步骤如下

1. 创建 Win32 窗口
2. 创建 DirectComposition 设备和关联窗口，获取渲染表面
3. 执行渲染逻辑

为了保持本文简洁，我将不在正文部分贴出非关键部分的代码，在本文末尾给出全部核心代码。本文的全部核心代码部分不到 200 行，适合一口气完成。本文也使用了到了一些库，为了防止大家不知道项目如何配置的，在本文末尾也给出整个项目全部代码和配置的下载方法

## 基础库

按照 .NET 的惯例，开始之前先安装基础库，安装之后的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <IsAotCompatible>true</IsAotCompatible>
    <PublishAot>true</PublishAot>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Vortice.Direct2D1" Version="3.8.2" />
    <PackageReference Include="Vortice.Direct3D11" Version="3.8.2" />
    <PackageReference Include="Vortice.DirectComposition" Version="3.8.2" />
    <PackageReference Include="Vortice.DXGI" Version="3.8.2" />
    <PackageReference Include="Vortice.Win32" Version="2.3.0" />

    <PackageReference Include="Microsoft.Windows.CsWin32" Version="0.3.257">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>

    <PackageReference Include="MicroCom.Runtime" Version="0.11.0" />

  </ItemGroup>

</Project>
```

## 创建 Win32 窗口

创建 Win32 窗口仅仅只是想拿到窗口句柄，不是本文重点，这里就忽略 CreateWindow 方法的实现

```csharp
        // 创建窗口
        HWND window = CreateWindow();
        // 显示窗口
        ShowWindow(window, SHOW_WINDOW_CMD.SW_NORMAL);
```

以上代码的 ShowWindow 是标准的 Win32 方法，由 CsWin32 库生成。定义如下

```csharp
		[DllImport("USER32.dll", ExactSpelling = true),DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
		[SupportedOSPlatform("windows5.0")]
		internal static extern winmdroot.Foundation.BOOL ShowWindow(winmdroot.Foundation.HWND hWnd, winmdroot.UI.WindowsAndMessaging.SHOW_WINDOW_CMD nCmdShow);
```

为了直接使用方法，在本文这里直接在命名空间引用静态类，代码如下

```csharp
using static Windows.Win32.PInvoke;
```

## 创建 DirectComposition 设备

先使用以下快速地代码创建 ID3D11Device 设备。正常来说，还是会尝试遍历获取显示适配器用来手动创建设备。本文这里使用的是比较不稳妥的简化写法。正确且常用的写法代码稍多，在本文这里 ID3D11Device 不是主角。如对此感兴趣，请参阅 [渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

```csharp
        var result = D3D11.D3D11CreateDevice(null, DriverType.Hardware, DeviceCreationFlags.BgraSupport,
            featureLevels: [], out ID3D11Device iD3D11Device, out var feature,
            out ID3D11DeviceContext iD3D11DeviceContext);
        result.CheckError();

        _ = feature;
        iD3D11DeviceContext.Dispose(); // 用不着就先释放。释放不代表立刻回收资源，只是表示业务层不需要用到，减少引用计数
```

在本文这里只用到了 ID3D11Device 设备，于是就选择立刻释放 ID3D11DeviceContext 的引用

拿到 ID3D11Device 设备，就可以调用 `DComp.DCompositionCreateDevice3` 创建 IDCompositionDevice 设备，代码如下

```csharp
        IDCompositionDevice compositionDevice = DComp.DCompositionCreateDevice3<IDCompositionDevice>(iD3D11Device);
```

这里的 `IDCompositionDevice` 应该就是 `IDirectCompositionDevice` 的缩写

## 对接窗口

通过 CreateTargetForHwnd 方法可以将 DirectComposition 关联到窗口上，代码如下

```csharp
        IDCompositionDevice compositionDevice = DComp.DCompositionCreateDevice3<IDCompositionDevice>(iD3D11Device);
        compositionDevice.CreateTargetForHwnd(window, topmost: true, out IDCompositionTarget compositionTarget);
```

这里的 CreateTargetForHwnd 的第二个参数比较迷惑，以上的 CreateTargetForHwnd 参数 topmost 不是值窗口置顶，而是：如果视觉树应显示在由 hwnd 参数指定的窗口子元素之上，则为 TRUE；否则，视觉树将显示在子元素之后。原文：

> TRUE if the visual tree should be displayed on top of the children of the window specified by the hwnd parameter; otherwise, the visual tree is displayed behind the children.

详细请看 <https://learn.microsoft.com/en-us/windows/win32/api/dcomp/nf-dcomp-idcompositiondevice-createtargetforhwnd>

## 创建视觉对象

创建视觉对象之前，需要获取当前窗口的尺寸，代码如下

```csharp
        RECT windowRect;
        PInvoke.GetClientRect(window, &windowRect);
        var clientSize = new SizeI(windowRect.right - windowRect.left, windowRect.bottom - windowRect.top);
```

创建视觉对象只需简单地调用 `IDCompositionDevice.CreateVisual` 方法，然而此时视觉对象还没有内容，可通过 `IDCompositionDevice.CreateVirtualSurface` 创建表面来作为内容，简单写法如下

```csharp
        IDCompositionVisual compositionVisual = compositionDevice.CreateVisual();
        IDCompositionVirtualSurface surface = compositionDevice.CreateVirtualSurface((uint) clientSize.Width,
            (uint) clientSize.Height, Format.B8G8R8A8_UNorm, AlphaMode.Premultiplied);
        compositionVisual.SetContent(surface);
```

创建表面时，可用 CreateVirtualSurface 或 CreateSurface 方法。两者不同的是 CreateVirtualSurface 创建的是稀疏表面，而 CreateSurface 是大数组（矩阵），在调用 BeginDraw 之前 IDCompositionVirtualSurface 不会分别空间，且 IDCompositionVirtualSurface 还能 Resize 重新设置大小。而 CreateSurface 则就不能。使用 CreateSurface 的例子如下

```csharp
var createSurfaceResult = compositionDevice.CreateSurface((uint) clientSize.Width,
            (uint) clientSize.Height, Format.B8G8R8A8_UNorm, AlphaMode.Premultiplied, out IDCompositionSurface? dCompositionSurface);
createSurfaceResult.CheckError();
```

创建表面需要传染颜色格式和对 AlphaMode 的处理，本文这里传入的是 Premultiplied 采用预乘方法。对 Premultiplied 简单来说就是最终输出的值里的 RGB 分量都乘以透明度。更多细节请参阅 [支持的像素格式和 Alpha 模式 - Win32 apps - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/direct2d/supported-pixel-formats-and-alpha-modes#about-premultiplied-and-straight-alpha-modes )

传入 Premultiplied 预乘时，会更多占用 DWM 的资源，在 4K 的全屏窗口上，对比 Premultiplied 预乘与忽略 AlphaMode 的性能，可以看到预乘会比忽略占用多非常多的 DWM 资源。如果自己的应用是无需窗口背景透明的，还请设置为忽略 AlphaMode 模式

## 配置视觉对象

完成视觉对象创建之后，可将此视觉对象设置为窗口的根内容。在 DirectComposition 里可以设置视觉树，一个视觉对象上可以添加很多个视觉对象，但只有其中一个可以成为 IDCompositionTarget 的 Root 视觉对象

```csharp
        compositionTarget.SetRoot(compositionVisual);
        compositionDevice.Commit(); // 非必须
```

## 对接 D2D 渲染

以上代码就完成了视觉对象的创建和在窗口上显示的基础逻辑。为了能够绘制漂亮的界面，在本文这里将和 D2D 进行对接

为了能够和 D2D 进行对接，需要给 D2D 一个绘制表面。从 IDCompositionVirtualSurface 或 IDCompositionSurface 表面调用 BeginDraw 方法，即可获取到 IDXGISurface 表面，从而让 D2D 在此表面上绘制，基础逻辑如下

```csharp
IDXGISurface dxgiSurface = surface.BeginDraw<IDXGISurface>(null, out var updateOffset);
```

再按照 D2D 的初始化方法，将 D2D 的 ID2D1RenderTarget 创建出来，代码如下

```csharp
        // 工厂创建只需一次
        Vortice.Direct2D1.ID2D1Factory1 d2DFactory = Vortice.Direct2D1.D2D1.D2D1CreateFactory<Vortice.Direct2D1.ID2D1Factory1>();

        // 以下循环为每一帧执行
        while (!_isMainWindowClosed)
        {
            using IDXGISurface dxgiSurface = surface.BeginDraw<IDXGISurface>(null, out var updateOffset);

            var renderTargetProperties = new Vortice.Direct2D1.RenderTargetProperties()
            {
                PixelFormat = new PixelFormat(Format.B8G8R8A8_UNorm, Vortice.DCommon.AlphaMode.Premultiplied),
                Type = Vortice.Direct2D1.RenderTargetType.Hardware,
            };

            using Vortice.Direct2D1.ID2D1RenderTarget d2D1RenderTarget =
                d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);

            Vortice.Direct2D1.ID2D1RenderTarget renderTarget = d2D1RenderTarget;

            ... // 忽略其他代码
        }
```

拿到 ID2D1RenderTarget 即可进行 D2D 的绘制，在本文这里使用了名为 D2DRenderDemo 的辅助类进行绘制，具体代码可以在后文找到

```csharp
        // 以下循环为每一帧执行
        while (!_isMainWindowClosed)
        {
            using IDXGISurface dxgiSurface = surface.BeginDraw<IDXGISurface>(null, out var updateOffset);

            var renderTargetProperties = new Vortice.Direct2D1.RenderTargetProperties()
            {
                PixelFormat = new PixelFormat(Format.B8G8R8A8_UNorm, Vortice.DCommon.AlphaMode.Premultiplied),
                Type = Vortice.Direct2D1.RenderTargetType.Hardware,
            };

            using Vortice.Direct2D1.ID2D1RenderTarget d2D1RenderTarget =
                d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);

            Vortice.Direct2D1.ID2D1RenderTarget renderTarget = d2D1RenderTarget;

            renderTarget.BeginDraw();

            // 在这里编写绘制逻辑
            renderTarget.Clear(new Color4(0f));
            d2DRenderDemo.Draw(renderTarget, clientSize);

            renderTarget.EndDraw();

            ... // 忽略其他代码
        }
```

当 D2D 绘制完成之后，需要调用 IDCompositionVirtualSurface 或 IDCompositionSurface 的 EndDraw 方法。再调用 IDCompositionDevice 的 Commit 方法将内容提交出去。当所有的窗口都完成绘制和 Commit 之后，调用 `IDCompositionDevice.WaitForCommitCompletion` 等待 DWM 消费。调用 `IDCompositionDevice.WaitForCommitCompletion` 约等于等待垂直同步，等待界面刷新

```csharp
            renderTarget.BeginDraw();

            // 在这里编写绘制逻辑
            renderTarget.Clear(new Color4(0f));
            d2DRenderDemo.Draw(renderTarget, clientSize);

            renderTarget.EndDraw();
            surface.EndDraw();

            compositionDevice.Commit();
            compositionDevice.WaitForCommitCompletion();
```

在本文这里，由于只有一个窗口，于是在 Commit 之后即可立刻调用 WaitForCommitCompletion 方法了

整个渲染代码，即每一帧跑的代码如下

```csharp
        Vortice.Direct2D1.ID2D1Factory1 d2DFactory = Vortice.Direct2D1.D2D1.D2D1CreateFactory<Vortice.Direct2D1.ID2D1Factory1>();

        var d2DRenderDemo = new D2DRenderDemo();

        while (!_isMainWindowClosed)
        {
            using IDXGISurface dxgiSurface = surface.BeginDraw<IDXGISurface>(null, out var updateOffset);
            _ = updateOffset;

            var renderTargetProperties = new Vortice.Direct2D1.RenderTargetProperties()
            {
                PixelFormat = new PixelFormat(Format.B8G8R8A8_UNorm, Vortice.DCommon.AlphaMode.Premultiplied),
                Type = Vortice.Direct2D1.RenderTargetType.Hardware,
            };

            using Vortice.Direct2D1.ID2D1RenderTarget d2D1RenderTarget =
                d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);

            Vortice.Direct2D1.ID2D1RenderTarget renderTarget = d2D1RenderTarget;

            renderTarget.BeginDraw();

            // 在这里编写绘制逻辑
            renderTarget.Clear(new Color4(0f));
            d2DRenderDemo.Draw(renderTarget, clientSize);

            renderTarget.EndDraw();
            surface.EndDraw();

            compositionDevice.Commit();
            compositionDevice.WaitForCommitCompletion();

            while (true)
            {
                var success = PInvoke.PeekMessage(out var message, window, 0, 0, PEEK_MESSAGE_REMOVE_TYPE.PM_REMOVE);
                if (!success)
                {
                    break;
                }

                PInvoke.TranslateMessage(&message);
                PInvoke.DispatchMessage(&message);
            }
        }
```

以上代码也跑了 PeekMessage 方法防止窗口未响应

在每次进入绘制的时候调用 `renderTarget.Clear(new Color4(0f));` 可以解决双缓存带来的闪烁问题，即界面内容被分别不同步地绘制到两个纹理表面上。通过 Clear 确保每次都是重新绘制，解决此问题

什么时候可以不做清理而进行绘制？进行某些性能优化的时候，且此时应该确保绘制同步。或再开一个表面纹理，通过同步的方式再将最终界面绘制

## 核心代码

核心的代码被我放在一个文件里面，代码如下

```csharp
using System;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using System.Runtime.Versioning;

using Vortice.DCommon;
using Vortice.Direct3D;
using Vortice.Direct3D11;
using Vortice.DirectComposition;
using Vortice.DXGI;
using Vortice.Mathematics;

using Windows.Win32;
using Windows.Win32.Foundation;
using Windows.Win32.Graphics.Gdi;
using Windows.Win32.UI.WindowsAndMessaging;

using AlphaMode = Vortice.DXGI.AlphaMode;
using D2D = Vortice.Direct2D1;

namespace DijallnemrecerkuCheberewhibair;

[SupportedOSPlatform("windows6.1")]
class DirectCompositionDemo
{
    public unsafe void Run()
    {
        var window = CreateWindow();
        PInvoke.ShowWindow(window, SHOW_WINDOW_CMD.SW_MAXIMIZE);

        var result = D3D11.D3D11CreateDevice(null, DriverType.Hardware, DeviceCreationFlags.BgraSupport,
            featureLevels: [], out ID3D11Device iD3D11Device, out var feature,
            out ID3D11DeviceContext iD3D11DeviceContext);
        result.CheckError();

        _ = feature;
        iD3D11DeviceContext.Dispose(); // 用不着就先释放。释放不代表立刻回收资源，只是表示业务层不需要用到，减少引用计数

        IDCompositionDevice compositionDevice = DComp.DCompositionCreateDevice3<IDCompositionDevice>(iD3D11Device);
        compositionDevice.CreateTargetForHwnd(window, topmost: true, out IDCompositionTarget compositionTarget);
        // 以上的 CreateTargetForHwnd 参数 topmost 不是值窗口置顶，而是：如果视觉树应显示在由 hwnd 参数指定的窗口子元素之上，则为 TRUE；否则，视觉树将显示在子元素之后
        // > TRUE if the visual tree should be displayed on top of the children of the window specified by the hwnd parameter; otherwise, the visual tree is displayed behind the children.
        // https://learn.microsoft.com/en-us/windows/win32/api/dcomp/nf-dcomp-idcompositiondevice-createtargetforhwnd

        // 创建视觉对象
        RECT windowRect;
        PInvoke.GetClientRect(window, &windowRect);
        var clientSize = new SizeI(windowRect.right - windowRect.left, windowRect.bottom - windowRect.top);

        IDCompositionVisual compositionVisual = compositionDevice.CreateVisual();
        IDCompositionVirtualSurface surface = compositionDevice.CreateVirtualSurface((uint) clientSize.Width,
            (uint) clientSize.Height, Format.B8G8R8A8_UNorm, AlphaMode.Premultiplied);

        // 创建 IDCompositionSurface 有两个方法，分别是 CreateVirtualSurface 和 CreateSurface 方法。两者不同的是 CreateVirtualSurface 创建的是稀疏表面，而 CreateSurface 是大数组（矩阵），在调用 BeginDraw 之前 IDCompositionVirtualSurface 不会分别空间，且 IDCompositionVirtualSurface 还能 Resize 重新设置大小。而 CreateSurface 则就不能
        //var createSurfaceResult = compositionDevice.CreateSurface((uint) clientSize.Width,
        //    (uint) clientSize.Height, Format.B8G8R8A8_UNorm, AlphaMode.Premultiplied, out IDCompositionSurface? dCompositionSurface);
        //createSurfaceResult.CheckError();
        //surface = dCompositionSurface;

        compositionVisual.SetContent(surface);

        compositionTarget.SetRoot(compositionVisual);
        compositionDevice.Commit();

        Vortice.Direct2D1.ID2D1Factory1 d2DFactory = Vortice.Direct2D1.D2D1.D2D1CreateFactory<Vortice.Direct2D1.ID2D1Factory1>();

        var d2DRenderDemo = new D2DRenderDemo();

        while (!_isMainWindowClosed)
        {
            using IDXGISurface dxgiSurface = surface.BeginDraw<IDXGISurface>(null, out var updateOffset);
            _ = updateOffset;

            var renderTargetProperties = new Vortice.Direct2D1.RenderTargetProperties()
            {
                PixelFormat = new PixelFormat(Format.B8G8R8A8_UNorm, Vortice.DCommon.AlphaMode.Premultiplied),
                Type = Vortice.Direct2D1.RenderTargetType.Hardware,
            };

            using Vortice.Direct2D1.ID2D1RenderTarget d2D1RenderTarget =
                d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);

            Vortice.Direct2D1.ID2D1RenderTarget renderTarget = d2D1RenderTarget;

            renderTarget.BeginDraw();

            // 在这里编写绘制逻辑
            renderTarget.Clear(new Color4(0f));
            d2DRenderDemo.Draw(renderTarget, clientSize);

            renderTarget.EndDraw();
            surface.EndDraw();

            compositionDevice.Commit();
            compositionDevice.WaitForCommitCompletion();

            while (true)
            {
                var success = PInvoke.PeekMessage(out var message, window, 0, 0, PEEK_MESSAGE_REMOVE_TYPE.PM_REMOVE);
                if (!success)
                {
                    break;
                }

                PInvoke.TranslateMessage(&message);
                PInvoke.DispatchMessage(&message);
            }
        }
    }

    private bool _isMainWindowClosed;

    private unsafe HWND CreateWindow()
    {
        PInvoke.DwmIsCompositionEnabled(out var compositionEnabled);

        if (!compositionEnabled)
        {
            Console.WriteLine($"无法启用透明窗口效果");
        }

        // [Windows 窗口样式 什么是 WS_EX_NOREDIRECTIONBITMAP 样式](https://blog.lindexi.com/post/Windows-%E7%AA%97%E5%8F%A3%E6%A0%B7%E5%BC%8F-%E4%BB%80%E4%B9%88%E6%98%AF-WS_EX_NOREDIRECTIONBITMAP-%E6%A0%B7%E5%BC%8F.html )
        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;

        var style = WNDCLASS_STYLES.CS_OWNDC | WNDCLASS_STYLES.CS_HREDRAW | WNDCLASS_STYLES.CS_VREDRAW;

        var defaultCursor = PInvoke.LoadCursor(
            new HINSTANCE(IntPtr.Zero), new PCWSTR(PInvoke.IDC_ARROW.Value));

        var className = $"lindexi-{Guid.NewGuid().ToString()}";
        var title = "The Title";
        _wndProcDelegate = new WNDPROC(WndProc); // 仅用于防止 GC 回收。详细请看 https://github.com/lindexi/lindexi_gd/pull/85
        fixed (char* pClassName = className)
        fixed (char* pTitle = title)
        {
            var wndClassEx = new WNDCLASSEXW
            {
                cbSize = (uint) Marshal.SizeOf<WNDCLASSEXW>(),
                style = style,
                lpfnWndProc = _wndProcDelegate,
                hInstance = new HINSTANCE(PInvoke.GetModuleHandle(null).DangerousGetHandle()),
                hCursor = defaultCursor,
                hbrBackground = new HBRUSH(IntPtr.Zero),
                lpszClassName = new PCWSTR(pClassName)
            };
            ushort atom = PInvoke.RegisterClassEx(in wndClassEx);

            var dwStyle = WINDOW_STYLE.WS_OVERLAPPEDWINDOW | WINDOW_STYLE.WS_VISIBLE;

            var windowHwnd = PInvoke.CreateWindowEx(
                exStyle,
                new PCWSTR((char*) atom),
                new PCWSTR(pTitle),
                dwStyle,
                0, 0, 1900, 1000,
                HWND.Null, HMENU.Null, HINSTANCE.Null, null);

            return windowHwnd;
        }
    }

    private WNDPROC? _wndProcDelegate;

    private LRESULT WndProc(HWND hwnd, uint message, WPARAM wParam, LPARAM lParam)
    {
        if (message == PInvoke.WM_CLOSE)
        {
            _isMainWindowClosed = true;
        }

        return PInvoke.DefWindowProc(hwnd, message, wParam, lParam);
    }
}

class D2DRenderDemo
{
    // 此为调试代码，绘制一些矩形条
    private List<D2DRenderInfo>? _renderList;

    public void OnReSize()
    {
        _renderList = null;
    }

    public void Draw(D2D.ID2D1RenderTarget renderTarget, SizeI clientSize)
    {
        var rectWeight = 10;
        var rectHeight = 20;

        var margin = 5;

        if (_renderList is null)
        {
            _renderList = new List<D2DRenderInfo>();

            for (int top = margin; top < clientSize.Height - rectHeight - margin; top += rectHeight + margin)
            {
                Rect rect = new Rect(margin, top, rectWeight, rectHeight);

                var color = new Color4(Random.Shared.NextSingle(), Random.Shared.NextSingle(),
                    Random.Shared.NextSingle());
                var step = Random.Shared.Next(1, 20);

                var renderInfo = new D2DRenderInfo(rect, step, color);
                _renderList.Add(renderInfo);
            }
        }

        for (var i = 0; i < _renderList.Count; i++)
        {
            var renderInfo = _renderList[i];
            using var brush = renderTarget.CreateSolidColorBrush(renderInfo.Color);

            renderTarget.FillRectangle(renderInfo.Rect, brush);

            var nextRect = renderInfo.Rect with
            {
                Width = renderInfo.Rect.Width + renderInfo.Step
            };

            if (nextRect.Width > clientSize.Width - margin * 2)
            {
                nextRect = nextRect with
                {
                    Width = rectWeight
                };
            }

            _renderList[i] = renderInfo with
            {
                Rect = nextRect
            };
        }
    }

    private readonly record struct D2DRenderInfo(Rect Rect, int Step, Color4 Color);
}
```

## 全部代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/daef15201848fb5c338f519d49f4879017590124/DirectX/DirectComposition/DijallnemrecerkuCheberewhibair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/daef15201848fb5c338f519d49f4879017590124/DirectX/DirectComposition/DijallnemrecerkuCheberewhibair) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin daef15201848fb5c338f519d49f4879017590124
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin daef15201848fb5c338f519d49f4879017590124
```

获取代码之后，进入 DirectX/DirectComposition/DijallnemrecerkuCheberewhibair 文件夹，即可获取到源代码

## 更多博客

渲染部分，关于 SharpDx 和 Vortice 的使用方法，包括入门级教程，请参阅：

- [渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
- [SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
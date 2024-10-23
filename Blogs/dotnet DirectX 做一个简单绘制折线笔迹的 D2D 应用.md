---
title: dotnet DirectX 做一个简单绘制折线笔迹的 D2D 应用
description: 本文将告诉大家如何从简单的控制台开始，使用 Vortice 辅助调用 Direct2D1 的功能，配合 WM_Pointer 消息，制作一个简单绘制触摸折线笔迹的 D2D 应用
tags: C#,D2D,DirectX,Vortice,Direct2D
category: 
---

<!-- CreateTime:2024/10/16 07:27:25 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->

前置博客： [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html )

本文属于 D2D 系列博客，更多 D2D 相关博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

在开始之前，我十分推荐大家先阅读 [分享一个在 dotnet 里使用 D2D 配合 AOT 开发小而美的应用开发经验](https://blog.lindexi.com/post/%E5%88%86%E4%BA%AB%E4%B8%80%E4%B8%AA%E5%9C%A8-dotnet-%E9%87%8C%E4%BD%BF%E7%94%A8-D2D-%E9%85%8D%E5%90%88-AOT-%E5%BC%80%E5%8F%91%E5%B0%8F%E8%80%8C%E7%BE%8E%E7%9A%84%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E7%BB%8F%E9%AA%8C.html ) 这篇博客，通过阅读此博客，可以让大家理解一些常用概念

本文实现的 D2D 应用，由于触摸数据是从 WM_Pointer 获取的，这就限制了在 Win7 下是不可用的

依然按照 [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html ) 博客提供的方法，从控制台开始创建 Win32 窗口，挂上交换链，初始化绘制上下文信息

本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

修改 NativeMethods.txt 文件，替换为如下代码，以下为本文例子代码所需要用到的所有 Win32 方法和常量等内容

```csharp
GetModuleHandle
PeekMessage
TranslateMessage
DispatchMessage
GetMessage
RegisterClassExW
DefWindowProc
LoadCursor
PostQuitMessage
CreateWindowExW
DestroyWindow
ShowWindow
GetSystemMetrics
AdjustWindowRectEx
GetClientRect
GetWindowRect
IDC_ARROW
WM_KEYDOWN
WM_KEYUP
WM_SYSKEYDOWN
WM_SYSKEYUP
WM_DESTROY
WM_QUIT
WM_PAINT
WM_CLOSE
WM_ACTIVATEAPP
VIRTUAL_KEY
GetPointerTouchInfo
ScreenToClient
GetPointerDeviceRects
ClientToScreen
WM_POINTERDOWN
WM_POINTERUPDATE
WM_POINTERUP
```

略过创建窗口和获取 D2D 上下文相关代码，如对这部分代码感兴趣，请参阅 [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html )

以下为已经获取到 ID2D1RenderTarget 的代码，继续添加对触摸数据的处理

```csharp
        // 在窗口的 dxgi 的平面上创建 D2D 的画布，如此即可让 D2D 绘制到窗口上
        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);
        d2D1RenderTarget.AntialiasMode = D2D.AntialiasMode.PerPrimitive;

        var renderTarget = d2D1RenderTarget;
```

定义一个基础数据结构，用于记录点的信息

```csharp
    readonly record struct Point2D(double X, double Y);
```

这些基础数据结构我在很多个项目里面都有定义，基础数学相关类型我也重复定义了很多次，且受限于我的数学知识，有些类型定义还是不正确的。好在我的伙伴 [SeWZC](https://github.com/SeWZC) 在 GitHub 上开源了数学库，这个数学库是按照正确的数学实现，实现了许多数学相关的类型。详细请看 <https://github.com/dotnet-campus/DotNetCampus.Numerics>

开个消息循环等待，防止控制台退出，顺带在此消息循环里面处理 Pointer 消息

```csharp
        // 开个消息循环等待
        Windows.Win32.UI.WindowsAndMessaging.MSG msg;
        while (true)
        {
            ...
        }
```

根据 [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%BB%8E-WM_POINTER-%E6%B6%88%E6%81%AF%E5%88%B0-Touch-%E4%BA%8B%E4%BB%B6.html ) 博客提供的方法进行对 WM_POINTER 消息的处理
<!-- [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18403860 ) -->

处理逻辑如下

```csharp
        // 开个消息循环等待
        Windows.Win32.UI.WindowsAndMessaging.MSG msg;
        while (true)
        {
            if (PeekMessage(out msg, default, 0, 0, PM_REMOVE) != false)
            {
                if (msg.message is PInvoke.WM_POINTERDOWN or PInvoke.WM_POINTERUPDATE or PInvoke.WM_POINTERUP)
                {
                    ...
                }
            }
        }
```

本文这里先不考虑多指，也不考虑多笔，直接就是相邻点连接为折线。先按照 [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%BB%8E-WM_POINTER-%E6%B6%88%E6%81%AF%E5%88%B0-Touch-%E4%BA%8B%E4%BB%B6.html ) 博客提供的方法对收到的 Pointer 点进行处理，这里将使用的是高精度的点

```csharp
                    var wparam = msg.wParam;
                    var pointerId = (uint)(ToInt32((IntPtr)wparam.Value) & 0xFFFF);
                    PInvoke.GetPointerTouchInfo(pointerId, out var info);
                    POINTER_INFO pointerInfo = info.pointerInfo;

                    global::Windows.Win32.Foundation.RECT pointerDeviceRect = default;
                    global::Windows.Win32.Foundation.RECT displayRect = default;

                    PInvoke.GetPointerDeviceRects(pointerInfo.sourceDevice, &pointerDeviceRect, &displayRect);

                    var point2D = new Point2D(
                        pointerInfo.ptHimetricLocationRaw.X / (double)pointerDeviceRect.Width * displayRect.Width +
                        displayRect.left,
                        pointerInfo.ptHimetricLocationRaw.Y / (double)pointerDeviceRect.Height * displayRect.Height +
                        displayRect.top);

                    point2D = new Point2D(point2D.X - screenTranslate.X, point2D.Y - screenTranslate.Y);

    private static int ToInt32(IntPtr ptr) => IntPtr.Size == 4 ? ptr.ToInt32() : (int)(ptr.ToInt64() & 0xffffffff);
```

以上拿到的 Point2D 就是 Pointer 消息收到的触摸点

为了简单起见，咱这里不获取历史点，只获取最新的点即可。将最新的点和上一个点连接做折线在屏幕上显示出来，如此即可获取很高的性能，很低的延迟

有双缓存的存在，推荐每次都是重新绘制，在实际使用中，即使每次都绘制整个界面，对整理的性能影响也几乎可以忽略。但为了方便演示，本文这里限制了点的数量，如果超过了一定数量，则将记录的部分点删掉

```csharp
        var pointList = new List<Point2D>();

        var screenTranslate = new Point(0, 0);
        PInvoke.ClientToScreen(hWnd, ref screenTranslate);

        // 开个消息循环等待
        Windows.Win32.UI.WindowsAndMessaging.MSG msg;
        while (true)
        {
            if (PeekMessage(out msg, default, 0, 0, PM_REMOVE) != false)
            {
                if (msg.message is PInvoke.WM_POINTERDOWN or PInvoke.WM_POINTERUPDATE or PInvoke.WM_POINTERUP)
                {
                    ...

                    point2D = new Point2D(point2D.X - screenTranslate.X, point2D.Y - screenTranslate.Y);

                    pointList.Add(point2D);
                    if (pointList.Count > 200)
                    {
                        // 不要让点太多，导致绘制速度太慢
                        pointList.RemoveRange(0, 100);
                    }

                    ...
                }
            }
        }
```


为了在屏幕显示出笔迹折线，这里需要先创建画刷。按照 [dotnet C# 使用 Vortice 创建 Direct2D1 的 ID2D1SolidColorBrush 纯色画刷](https://blog.lindexi.com/post/dotnet-C-%E4%BD%BF%E7%94%A8-Vortice-%E5%88%9B%E5%BB%BA-Direct2D1-%E7%9A%84-ID2D1SolidColorBrush-%E7%BA%AF%E8%89%B2%E7%94%BB%E5%88%B7.html ) 博客介绍的方法创建简单的纯色画刷，代码如下

```csharp
                    var color = new Color4(0xFF0000FF);
                    using var brush = renderTarget.CreateSolidColorBrush(color);
```

接着开始构成折线，开始之前和结束之后别忘了调用 ` renderTarget.BeginDraw();` 和 `renderTarget.EndDraw();` 方法

```csharp
                    renderTarget.BeginDraw();
                    renderTarget.AntialiasMode = AntialiasMode.Aliased;

                    renderTarget.Clear(new Color4(0xFFFFFFFF));

                    for (var i = 1; i < pointList.Count; i++)
                    {
                        var previousPoint = pointList[i - 1];
                        var currentPoint = pointList[i];

                        renderTarget.DrawLine(new Vector2((float)previousPoint.X, (float)previousPoint.Y),
                            new Vector2((float)currentPoint.X, (float)currentPoint.Y), brush, 5);
                    }

                    renderTarget.EndDraw();
```

以上代码通过多次 DrawLine 的方式完成笔迹折线的。完成绘制之后，调用一下 `swapChain.Present` 切换交换链，从而在界面显示笔迹折线

```csharp
                    renderTarget.EndDraw();
                    swapChain.Present(1, DXGI.PresentFlags.None);
                    // 等待刷新
                    d3D11DeviceContext.Flush();
```

以上就是使用 Vortice 辅助调用 Direct2D1 的功能，配合 WM_Pointer 消息，制作一个简单绘制触摸折线笔迹的 D2D 应用的核心逻辑

本文的例子代码非常简单，可以全部在一个 Program.cs 文件完成，所有代码如下

```csharp
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;
using System.Runtime.Versioning;
using Windows.Win32.Foundation;
using Windows.Win32.UI.WindowsAndMessaging;
using static Windows.Win32.PInvoke;
using static Windows.Win32.UI.WindowsAndMessaging.PEEK_MESSAGE_REMOVE_TYPE;
using static Windows.Win32.UI.WindowsAndMessaging.WNDCLASS_STYLES;
using static Windows.Win32.UI.WindowsAndMessaging.WINDOW_STYLE;
using static Windows.Win32.UI.WindowsAndMessaging.WINDOW_EX_STYLE;
using static Windows.Win32.UI.WindowsAndMessaging.SYSTEM_METRICS_INDEX;
using static Windows.Win32.UI.WindowsAndMessaging.SHOW_WINDOW_CMD;
using Vortice.Mathematics;
using AlphaMode = Vortice.DXGI.AlphaMode;
using D3D = Vortice.Direct3D;
using D3D11 = Vortice.Direct3D11;
using DXGI = Vortice.DXGI;
using D2D = Vortice.Direct2D1;
using System.Drawing;
using Vortice.Direct2D1;
using System.Numerics;
using Windows.Win32;
using Windows.Win32.UI.Input.Pointer;

namespace QalberegejeaJawchejoleawerejea;

class Program
{
    // 设置可以支持 Win7 和以上版本。如果用到 WinRT 可以设置为支持 win10 和以上。这个特性只是给 VS 看的，没有实际影响运行的逻辑
    [SupportedOSPlatform("Windows7.0")]
    static unsafe void Main(string[] args)
    {
        // 准备创建窗口
        // 使用 Win32 创建窗口需要很多参数，这些参数系列不是本文的重点，还请自行了解
        SizeI clientSize = new SizeI(1000, 600);

        // 窗口标题
        var title = "QalberegejeaJawchejoleawerejea";
        var windowClassName = "lindexi doubi";

        // 窗口样式，窗口样式含义请执行参阅官方文档，样式只要不离谱，自己随便写，影响不大
        WINDOW_STYLE style = WS_CAPTION |
                             WS_SYSMENU |
                             WS_MINIMIZEBOX |
                             WS_CLIPSIBLINGS |
                             WS_BORDER |
                             WS_DLGFRAME |
                             WS_THICKFRAME |
                             WS_GROUP |
                             WS_TABSTOP |
                             WS_SIZEBOX;

        var rect = new RECT
        {
            right = clientSize.Width,
            bottom = clientSize.Height
        };

        // Adjust according to window styles
        AdjustWindowRectEx(&rect, style, false, WS_EX_APPWINDOW);

        // 决定窗口在哪显示，这个不影响大局
        int x = 0;
        int y = 0;
        int windowWidth = rect.right - rect.left;
        int windowHeight = rect.bottom - rect.top;

        // 随便，放在屏幕中间好了。多个显示器？忽略
        int screenWidth = GetSystemMetrics(SM_CXSCREEN);
        int screenHeight = GetSystemMetrics(SM_CYSCREEN);

        x = (screenWidth - windowWidth) / 2;
        y = (screenHeight - windowHeight) / 2;

        var hInstance = GetModuleHandle((string?)null);

        fixed (char* lpszClassName = windowClassName)
        {
            PCWSTR szCursorName = new((char*)IDC_ARROW);

            var wndClassEx = new WNDCLASSEXW
            {
                cbSize = (uint)Unsafe.SizeOf<WNDCLASSEXW>(),
                style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC,
                // 核心逻辑，设置消息循环
                lpfnWndProc = new WNDPROC(WndProc),
                hInstance = (HINSTANCE)hInstance.DangerousGetHandle(),
                hCursor = LoadCursor((HINSTANCE)IntPtr.Zero, szCursorName),
                hbrBackground = (Windows.Win32.Graphics.Gdi.HBRUSH)IntPtr.Zero,
                hIcon = (HICON)IntPtr.Zero,
                lpszClassName = lpszClassName
            };

            ushort atom = RegisterClassEx(wndClassEx);

            if (atom == 0)
            {
                throw new InvalidOperationException(
                    $"Failed to register window class. Error: {Marshal.GetLastWin32Error()}"
                );
            }
        }

        // 创建窗口
        var hWnd = CreateWindowEx
        (
            WS_EX_APPWINDOW,
            windowClassName,
            title,
            style,
            x,
            y,
            windowWidth,
            windowHeight,
            hWndParent: default,
            hMenu: default,
            hInstance: default,
            lpParam: null
        );

        // 创建完成，那就显示
        ShowWindow(hWnd, SW_NORMAL);
        RECT windowRect;
        GetClientRect(hWnd, &windowRect);
        clientSize = new SizeI(windowRect.right - windowRect.left, windowRect.bottom - windowRect.top);

        // 开始创建工厂创建 D3D 的逻辑
        var dxgiFactory2 = DXGI.DXGI.CreateDXGIFactory1<DXGI.IDXGIFactory2>();

        var hardwareAdapter = GetHardwareAdapter(dxgiFactory2)
            // 这里 ToList 只是想列出所有的 IDXGIAdapter1 在实际代码里，大部分都是获取第一个
            .ToList().FirstOrDefault();
        if (hardwareAdapter == null)
        {
            throw new InvalidOperationException("Cannot detect D3D11 adapter");
        }
        else
        {
            Console.WriteLine($"使用显卡 {hardwareAdapter.Description1.Description}");
        }

        // 功能等级
        // [C# 从零开始写 SharpDx 应用 聊聊功能等级](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E8%81%8A%E8%81%8A%E5%8A%9F%E8%83%BD%E7%AD%89%E7%BA%A7.html)
        D3D.FeatureLevel[] featureLevels = new[]
        {
            D3D.FeatureLevel.Level_11_1,
            D3D.FeatureLevel.Level_11_0,
            D3D.FeatureLevel.Level_10_1,
            D3D.FeatureLevel.Level_10_0,
            D3D.FeatureLevel.Level_9_3,
            D3D.FeatureLevel.Level_9_2,
            D3D.FeatureLevel.Level_9_1,
        };

        DXGI.IDXGIAdapter1 adapter = hardwareAdapter;
        D3D11.DeviceCreationFlags creationFlags = D3D11.DeviceCreationFlags.BgraSupport;
        var result = D3D11.D3D11.D3D11CreateDevice
        (
            adapter,
            D3D.DriverType.Unknown,
            creationFlags,
            featureLevels,
            out D3D11.ID3D11Device d3D11Device, out D3D.FeatureLevel featureLevel,
            out D3D11.ID3D11DeviceContext d3D11DeviceContext
        );

        if (result.Failure)
        {
            // 如果失败了，那就不指定显卡，走 WARP 的方式
            // http://go.microsoft.com/fwlink/?LinkId=286690
            result = D3D11.D3D11.D3D11CreateDevice(
                IntPtr.Zero,
                D3D.DriverType.Warp,
                creationFlags,
                featureLevels,
                out d3D11Device, out featureLevel, out d3D11DeviceContext);

            // 如果失败，就不能继续
            result.CheckError();
        }

        // 大部分情况下，用的是 ID3D11Device1 和 ID3D11DeviceContext1 类型
        // 从 ID3D11Device 转换为 ID3D11Device1 类型
        var d3D11Device1 = d3D11Device.QueryInterface<D3D11.ID3D11Device1>();
        var d3D11DeviceContext1 = d3D11DeviceContext.QueryInterface<D3D11.ID3D11DeviceContext1>();

        // 后续还要创建 D2D 设备，就先不考虑释放咯
        //// 转换完成，可以减少对 ID3D11Device1 的引用计数
        //// 调用 Dispose 不会释放掉刚才申请的 D3D 资源，只是减少引用计数
        //d3D11Device.Dispose();
        //d3D11DeviceContext.Dispose();

        // 创建设备，接下来就是关联窗口和交换链
        DXGI.Format colorFormat = DXGI.Format.B8G8R8A8_UNorm;

        const int FrameCount = 2;

        DXGI.SwapChainDescription1 swapChainDescription = new()
        {
            Width = (uint)clientSize.Width,
            Height = (uint)clientSize.Height,
            Format = colorFormat,
            BufferCount = FrameCount,
            BufferUsage = DXGI.Usage.RenderTargetOutput,
            SampleDescription = DXGI.SampleDescription.Default,
            Scaling = DXGI.Scaling.Stretch,
            SwapEffect = DXGI.SwapEffect.FlipSequential,
            AlphaMode = AlphaMode.Ignore,
            // https://learn.microsoft.com/zh-cn/windows/win32/api/dxgi/nf-dxgi-idxgiswapchain-present
            // 可变刷新率显示 启用撕裂是可变刷新率显示器的要求
            //Flags = DXGI.SwapChainFlags.AllowTearing,
        };
        // 设置是否全屏
        DXGI.SwapChainFullscreenDescription fullscreenDescription = new DXGI.SwapChainFullscreenDescription
        {
            Windowed = true,
        };

        // 给创建出来的窗口挂上交换链
        DXGI.IDXGISwapChain1 swapChain =
            dxgiFactory2.CreateSwapChainForHwnd(d3D11Device1, hWnd, swapChainDescription, fullscreenDescription);

        // 不要被按下 alt+enter 进入全屏
        dxgiFactory2.MakeWindowAssociation(hWnd, DXGI.WindowAssociationFlags.IgnoreAltEnter);

        D3D11.ID3D11Texture2D backBufferTexture = swapChain.GetBuffer<D3D11.ID3D11Texture2D>(0);

        // 获取到 dxgi 的平面，这个屏幕就约等于窗口渲染内容
        DXGI.IDXGISurface dxgiSurface = backBufferTexture.QueryInterface<DXGI.IDXGISurface>();

        // 对接 D2D 需要创建工厂
        D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();

        // 方法1：
        var renderTargetProperties = new D2D.RenderTargetProperties(Vortice.DCommon.PixelFormat.Premultiplied);

        // 在窗口的 dxgi 的平面上创建 D2D 的画布，如此即可让 D2D 绘制到窗口上
        D2D.ID2D1RenderTarget d2D1RenderTarget =
            d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);
        d2D1RenderTarget.AntialiasMode = D2D.AntialiasMode.PerPrimitive;

        var renderTarget = d2D1RenderTarget;

        // 方法2：
        // 创建 D2D 设备，通过设置 ID2D1DeviceContext 的 Target 输出为 dxgiSurface 从而让 ID2D1DeviceContext 渲染内容渲染到窗口上
        // 如 https://learn.microsoft.com/en-us/windows/win32/direct2d/images/devicecontextdiagram.png 图
        // 获取 DXGI 设备，用来创建 D2D 设备
        //DXGI.IDXGIDevice dxgiDevice = d3D11Device.QueryInterface<DXGI.IDXGIDevice>();
        //ID2D1Device d2dDevice = d2DFactory.CreateDevice(dxgiDevice);
        //ID2D1DeviceContext d2dDeviceContext = d2dDevice.CreateDeviceContext();

        //ID2D1Bitmap1 d2dBitmap = d2dDeviceContext.CreateBitmapFromDxgiSurface(dxgiSurface);
        //d2dDeviceContext.Target = d2dBitmap;

        //var renderTarget = d2dDeviceContext;

        var pointList = new List<Point2D>();

        var screenTranslate = new Point(0, 0);
        PInvoke.ClientToScreen(hWnd, ref screenTranslate);

        // 开个消息循环等待
        Windows.Win32.UI.WindowsAndMessaging.MSG msg;
        while (true)
        {
            if (PeekMessage(out msg, default, 0, 0, PM_REMOVE) != false)
            {
                if (msg.message is PInvoke.WM_POINTERDOWN or PInvoke.WM_POINTERUPDATE or PInvoke.WM_POINTERUP)
                {
                    var wparam = msg.wParam;
                    var pointerId = (uint)(ToInt32((IntPtr)wparam.Value) & 0xFFFF);
                    PInvoke.GetPointerTouchInfo(pointerId, out var info);
                    POINTER_INFO pointerInfo = info.pointerInfo;

                    global::Windows.Win32.Foundation.RECT pointerDeviceRect = default;
                    global::Windows.Win32.Foundation.RECT displayRect = default;

                    PInvoke.GetPointerDeviceRects(pointerInfo.sourceDevice, &pointerDeviceRect, &displayRect);

                    var point2D = new Point2D(
                        pointerInfo.ptHimetricLocationRaw.X / (double)pointerDeviceRect.Width * displayRect.Width +
                        displayRect.left,
                        pointerInfo.ptHimetricLocationRaw.Y / (double)pointerDeviceRect.Height * displayRect.Height +
                        displayRect.top);

                    point2D = new Point2D(point2D.X - screenTranslate.X, point2D.Y - screenTranslate.Y);

                    pointList.Add(point2D);
                    if (pointList.Count > 200)
                    {
                        // 不要让点太多，导致绘制速度太慢
                        pointList.RemoveRange(0, 100);
                    }

                    var color = new Color4(0xFF0000FF);
                    using var brush = renderTarget.CreateSolidColorBrush(color);

                    renderTarget.BeginDraw();
                    renderTarget.AntialiasMode = AntialiasMode.Aliased;

                    renderTarget.Clear(new Color4(0xFFFFFFFF));

                    for (var i = 1; i < pointList.Count; i++)
                    {
                        var previousPoint = pointList[i - 1];
                        var currentPoint = pointList[i];

                        renderTarget.DrawLine(new Vector2((float)previousPoint.X, (float)previousPoint.Y),
                            new Vector2((float)currentPoint.X, (float)currentPoint.Y), brush, 5);
                    }

                    renderTarget.EndDraw();
                    swapChain.Present(1, DXGI.PresentFlags.None);
                    // 等待刷新
                    d3D11DeviceContext.Flush();
                }

                _ = TranslateMessage(&msg);
                _ = DispatchMessage(&msg);

                if (msg.message is WM_QUIT or WM_CLOSE)
                {
                    return;
                }
            }
        }
    }

    private static int ToInt32(IntPtr ptr) => IntPtr.Size == 4 ? ptr.ToInt32() : (int)(ptr.ToInt64() & 0xffffffff);

    private static IEnumerable<DXGI.IDXGIAdapter1> GetHardwareAdapter(DXGI.IDXGIFactory2 factory)
    {
        DXGI.IDXGIFactory6? factory6 = factory.QueryInterfaceOrNull<DXGI.IDXGIFactory6>();
        if (factory6 != null)
        {
            // 先告诉系统，要高性能的显卡
            for (uint adapterIndex = 0;
                 factory6.EnumAdapterByGpuPreference(adapterIndex, DXGI.GpuPreference.Unspecified,
                     out DXGI.IDXGIAdapter1? adapter).Success;
                 adapterIndex++)
            {
                if (adapter == null)
                {
                    continue;
                }

                DXGI.AdapterDescription1 desc = adapter.Description1;

                if ((desc.Flags & DXGI.AdapterFlags.Software) != DXGI.AdapterFlags.None)
                {
                    // Don't select the Basic Render Driver adapter.
                    adapter.Dispose();
                    continue;
                }

                Console.WriteLine($"枚举到 {adapter.Description1.Description} 显卡");
                yield return adapter;
            }

            factory6.Dispose();
        }

        // 如果枚举不到，那系统返回啥都可以
        for (uint adapterIndex = 0;
             factory.EnumAdapters1(adapterIndex, out DXGI.IDXGIAdapter1? adapter).Success;
             adapterIndex++)
        {
            DXGI.AdapterDescription1 desc = adapter.Description1;

            if ((desc.Flags & DXGI.AdapterFlags.Software) != DXGI.AdapterFlags.None)
            {
                // Don't select the Basic Render Driver adapter.
                adapter.Dispose();

                continue;
            }

            Console.WriteLine($"枚举到 {adapter.Description1.Description} 显卡");
            yield return adapter;
        }
    }

    private static LRESULT WndProc(HWND hWnd, uint message, WPARAM wParam, LPARAM lParam)
    {
        return DefWindowProc(hWnd, message, wParam, lParam);
    }

    readonly record struct Point2D(double X, double Y);
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b5109772231d99b403092ce9d29bcbcf0f23b2e2/DirectX/D2D/QalberegejeaJawchejoleawerejea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b5109772231d99b403092ce9d29bcbcf0f23b2e2/DirectX/D2D/QalberegejeaJawchejoleawerejea) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b5109772231d99b403092ce9d29bcbcf0f23b2e2
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b5109772231d99b403092ce9d29bcbcf0f23b2e2
```

获取代码之后，进入 DirectX/D2D/QalberegejeaJawchejoleawerejea 文件夹，即可获取到源代码。欢迎大家拉下来代码跑跑看性能，这个简单的应用能够追得上 WPF 的笔迹应用的性能。本文介绍的这个应用还不能达到 D2D 的最优性能，还有很多优化空间。预计极限性能，笔迹的延迟能和 WPF 追平，部分特殊情况下能够超越 WPF 的性能。本文绘制的笔迹比较粗糙，只是简单的折线，没有带任何笔迹路径平滑和边缘采样优化。如果大家对从触摸收到的点集转换为笔迹路径好奇，请参阅 [WPF 笔迹算法 从点集转笔迹轮廓](https://blog.lindexi.com/post/WPF-%E7%AC%94%E8%BF%B9%E7%AE%97%E6%B3%95-%E4%BB%8E%E7%82%B9%E9%9B%86%E8%BD%AC%E7%AC%94%E8%BF%B9%E8%BD%AE%E5%BB%93.html )

更多渲染和触摸博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

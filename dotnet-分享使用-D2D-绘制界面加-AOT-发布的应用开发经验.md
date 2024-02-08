
# dotnet 分享使用 D2D 绘制界面加 AOT 发布的应用开发经验

这是我用不到 370 行代码，从零开始控制台创建 Win32 窗口，再挂上交换链，在窗口上使用 D2D 绘制界面内容。最后使用 AOT 方式发布的测试应用。成品文件体积不超过 10MB 且运行内存稳定在 60MB 以内，满帧率运行但 CPU 近乎不动

<!--more-->


<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

整个测试应用采用了 .NET 8 的框架，用于更好的支持 AOT 发布

使用了 Vortice 系列库用于对 DirectX 的封装，方便让编写调用 DirectX 的代码

使用了 Microsoft.Windows.CsWin32 方便进行 Win32 方法的调用

所有的代码都写在 Program.cs 文件里面，代码长度不到 370 行，更有趣的是，可以强行算是都写在 Main 方法里面，由 Main 方法以及放在 Main 方法里面的局部方法构成。整体实现非常简单。我将会在本文末尾告诉大家本文的代码的下载方法

本文仅仅是分享我的开发经验，不包含 DirectX 的前置知识。如果不熟悉 D2D 和 DirectX 还请以看着玩的心态阅读本文

一开始采用了 [DirectX 使用 Vortice 从零开始控制台创建 Direct2D1 窗口修改颜色](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html ) 和 [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html ) 博客提供的方法搭建了基础的应用框架

为了让界面更加的丰富，我准备在界面添加多个圆形。然后为了让界面动起来，我添加了名为 DrawingInfo 的结构体，用于存放每个圆形的坐标和大小等信息

```csharp
readonly record struct DrawingInfo(System.Numerics.Vector2 Offset, Size Size, D2D.ID2D1SolidColorBrush Brush);
```

先在绘制的循环外对 DrawingInfo 进行随机设置值

```csharp
            var ellipseInfoList = new List<DrawingInfo>();
            for (int i = 0; i < 3000; i++)
            {
                // 随意创建颜色
                var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255));
                D2D.ID2D1SolidColorBrush brush = renderTarget.CreateSolidColorBrush(color);
                ellipseInfoList.Add(new DrawingInfo(new System.Numerics.Vector2(Random.Shared.Next(clientSize.Width), Random.Shared.Next(clientSize.Height)), new Size(Random.Shared.Next(10, 100)), brush));
            }
```

进入循环之后，再每次修改 Offset 的值，这样就可以让每次绘制的圆形动起来

```csharp
            while (true)
            {
                // 开始绘制逻辑
                renderTarget.BeginDraw();

                // 清空画布
                renderTarget.Clear(new Color4(0xFF, 0xFF, 0xFF));

                // 在下面绘制漂亮的界面

                for (var i = 0; i < ellipseInfoList.Count; i++)
                {
                    var drawingInfo = ellipseInfoList[i];
                    var vector2 = drawingInfo.Offset;
                    vector2.X += Random.Shared.Next(200) - 100;
                    vector2.Y += Random.Shared.Next(200) - 100;

                    while (vector2.X < 100 || vector2.X > clientSize.Width - 100)
                    {
                        vector2.X = Random.Shared.Next(clientSize.Width);
                    }

                    while (vector2.Y < 100 || vector2.Y > clientSize.Height - 100)
                    {
                        vector2.Y = Random.Shared.Next(clientSize.Height);
                    }

                    ellipseInfoList[i] = drawingInfo with { Offset = vector2 };
                    
                    // 忽略其他代码
                }

                // 忽略其他代码
            }
```

以上的修改坐标代码只是为了让圆形每次都在其附近移动

附带就在里层循环将每个圆形绘制，代码如下

```csharp
                // 在下面绘制漂亮的界面

                for (var i = 0; i < ellipseInfoList.Count; i++)
                {
                    // 忽略其他代码
                    renderTarget.FillEllipse(new D2D.Ellipse(vector2, drawingInfo.Size.Width, drawingInfo.Size.Height), drawingInfo.Brush);
                }
```

大概的改动如此，接下来咱需要改造一下 csproj 项目文件，让此项目可以构建出 AOT 版本的应用

先修改 TargetFramework 为 net8.0 使用 .NET 8 可以更好构建 AOT 应用

```xml
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
```

接着为了减少不断提示的平台警告，添加以下代码忽略 CA1416 警告

```xml
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <NoWarn>CA1416</NoWarn>
  </PropertyGroup>
```

接着再添加 PublishAot 属性，这样调用发布命令之后，就可以自动创建 AOT 应用的文件

```xml
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <NoWarn>CA1416</NoWarn>
    <PublishAot>true</PublishAot>
  </PropertyGroup>
```

此时运行起来将不会成功，将会提示大概如下的错误

```
Unhandled Exception: System.MissingMethodException: No parameterless constructor defined for type 'Vortice.DXGI.IDXGIFactory2'.
   at System.ActivatorImplementation.CreateInstance(Type, BindingFlags, Binder, Object[], CultureInfo, Object[]) + 0x348
   at SharpGen.Runtime.MarshallingHelpers.FromPointer[T](IntPtr) + 0x8c
   at Vortice.DXGI.DXGI.CreateDXGIFactory1[T]() + 0x55
   at Program.<<Main>$>g__CreateD2D|0_2(Program.<>c__DisplayClass0_0&) + 0x90
   at Program.<Main>$(String[] args) + 0x23e
   at CedageawhakairnerewhalNaibiferenagifee!<BaseAddress>+0x17a3c0
```

或者是如下的错误

```
Unhandled Exception: System.MissingMethodException: No parameterless constructor defined for type 'Vortice.Direct3D11.ID3D11Device1'.
   at System.ActivatorImplementation.CreateInstance(Type, BindingFlags, Binder, Object[], CultureInfo, Object[]) + 0x348
   at SharpGen.Runtime.MarshallingHelpers.FromPointer[T](IntPtr) + 0x8c
   at SharpGen.Runtime.ComObject.QueryInterface[T]() + 0x64
   at Program.<<Main>$>g__CreateD2D|0_2(Program.<>c__DisplayClass0_0&) + 0x1c7
   at Program.<Main>$(String[] args) + 0x23e
   at CedageawhakairnerewhalNaibiferenagifee!<BaseAddress>+0x335cf0
```

这是因为这些引用的库里面的类型在 AOT 的裁剪过程被丢掉

修复的方法很简单，那就是将 Vortice 添加到 TrimmerRootAssembly 里面，防止在 AOT 过程被裁剪

```xml
  <ItemGroup>
    <TrimmerRootAssembly Include="Vortice.Win32"/>
    <TrimmerRootAssembly Include="Vortice.DXGI"/>
    <TrimmerRootAssembly Include="Vortice.Direct3D11"/>
    <TrimmerRootAssembly Include="Vortice.Direct2D1"/>
    <TrimmerRootAssembly Include="Vortice.D3DCompiler"/>
    <TrimmerRootAssembly Include="Vortice.DirectX"/>
    <TrimmerRootAssembly Include="Vortice.Mathematics"/>
  </ItemGroup>
```

修改之后的 csproj 代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <PublishAot>true</PublishAot>
    <NoWarn>CA1416</NoWarn>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Vortice.Direct2D1" Version="2.1.32" />
    <PackageReference Include="Vortice.Direct3D11" Version="2.1.32" />
    <PackageReference Include="Vortice.DirectX" Version="2.1.32" />
    <PackageReference Include="Vortice.D3DCompiler" Version="2.1.32" />
    <PackageReference Include="Vortice.Win32" Version="1.6.2" />
    <PackageReference Include="Microsoft.Windows.CsWin32" PrivateAssets="all" Version="0.2.63-beta" />
  </ItemGroup>
  <ItemGroup>
    <TrimmerRootAssembly Include="Vortice.Win32"/>
    <TrimmerRootAssembly Include="Vortice.DXGI"/>
    <TrimmerRootAssembly Include="Vortice.Direct3D11"/>
    <TrimmerRootAssembly Include="Vortice.Direct2D1"/>
    <TrimmerRootAssembly Include="Vortice.D3DCompiler"/>
    <TrimmerRootAssembly Include="Vortice.DirectX"/>
    <TrimmerRootAssembly Include="Vortice.Mathematics"/>
  </ItemGroup>
</Project>
```

完成以上配置之后，即可使用命令行 dotnet publish 将项目进行发布，如果在发布的控制台可以看到 Generating native code 输出，那就证明配置正确，正在构建 AOT 文件

完成构建之后，即可在 `bin\Release\net8.0\win-x64\publish` 文件夹找到构建输出的文件，在我这里看到的输出文件大小大概在 10MB 以下，大家可以尝试使用本文末尾的方法拉取我的代码自己构建一下，试试效果

运行起来的任务管理器所见内存大小大约是 30MB 左右，通过 VMMap 工具查看 WorkingSet 和 Private Bytes 都在 60MB 以内。虽然 Committed 的内存高达 300MB 但是绝大部分都是 Image 共享部分占用内存，如显卡驱动等部分的占用，这部分占用大约在 250MB 以上，实际的 Image 的 private 的占用不到 10MB 大小

我认为这个技术可以用来制作一些小而美的工具，甚至是不用考虑 x86 的，只需考虑 x64 的机器上运行的应用的安装包制作程序。要是拿着 D2D 绘制的界面去当安装包的界面，那估计安装包行业会卷起来

以下是所有的代码

```csharp
using D3D = Vortice.Direct3D;
using D3D11 = Vortice.Direct3D11;
using DXGI = Vortice.DXGI;
using D2D = Vortice.Direct2D1;

using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;
using Windows.Win32.Foundation;
using Windows.Win32.UI.WindowsAndMessaging;
using static Windows.Win32.PInvoke;
using static Windows.Win32.UI.WindowsAndMessaging.PEEK_MESSAGE_REMOVE_TYPE;
using static Windows.Win32.UI.WindowsAndMessaging.WNDCLASS_STYLES;
using static Windows.Win32.UI.WindowsAndMessaging.WINDOW_STYLE;
using static Windows.Win32.UI.WindowsAndMessaging.WINDOW_EX_STYLE;
using static Windows.Win32.UI.WindowsAndMessaging.SYSTEM_METRICS_INDEX;
using static Windows.Win32.UI.WindowsAndMessaging.SHOW_WINDOW_CMD;
using Vortice.DCommon;
using Vortice.Mathematics;
using AlphaMode = Vortice.DXGI.AlphaMode;
using System.Diagnostics;

unsafe
{
    SizeI clientSize = new SizeI(1000, 1000);

    // 窗口标题
    var title = "lindexi D2D AOT";
    var windowClassName = title;

    WINDOW_STYLE style = WS_CAPTION | WS_SYSMENU | WS_MINIMIZEBOX | WS_CLIPSIBLINGS | WS_BORDER | WS_DLGFRAME | WS_THICKFRAME | WS_GROUP | WS_TABSTOP | WS_SIZEBOX;

    var rect = new RECT
    {
        right = clientSize.Width,
        bottom = clientSize.Height
    };

    AdjustWindowRectEx(&rect, style, false, WS_EX_APPWINDOW);

    int x = 0;
    int y = 0;
    int windowWidth = rect.right - rect.left;
    int windowHeight = rect.bottom - rect.top;

    // 随便，放在屏幕中间好了。多个显示器？忽略
    int screenWidth = GetSystemMetrics(SM_CXSCREEN);
    int screenHeight = GetSystemMetrics(SM_CYSCREEN);

    x = (screenWidth - windowWidth) / 2;
    y = (screenHeight - windowHeight) / 2;

    var hInstance = GetModuleHandle((string) null);

    fixed (char* lpszClassName = windowClassName)
    {
        PCWSTR szCursorName = new((char*) IDC_ARROW);

        var wndClassEx = new WNDCLASSEXW
        {
            cbSize = (uint) Unsafe.SizeOf<WNDCLASSEXW>(),
            style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC,
            // 核心逻辑，设置消息循环
            lpfnWndProc = new WNDPROC(WndProc),
            hInstance = (HINSTANCE) hInstance.DangerousGetHandle(),
            hCursor = LoadCursor((HINSTANCE) IntPtr.Zero, szCursorName),
            hbrBackground = (Windows.Win32.Graphics.Gdi.HBRUSH) IntPtr.Zero,
            hIcon = (HICON) IntPtr.Zero,
            lpszClassName = lpszClassName
        };

        ushort atom = RegisterClassEx(wndClassEx);

        if (atom == 0)
        {
            throw new InvalidOperationException($"Failed to register window class. Error: {Marshal.GetLastWin32Error()}");
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

    CreateD2D();

    // 开个消息循环等待
    Windows.Win32.UI.WindowsAndMessaging.MSG msg;
    while (true)
    {
        if (GetMessage(out msg, hWnd, 0, 0) != false)
        {
            _ = TranslateMessage(&msg);
            _ = DispatchMessage(&msg);

            if (msg.message is WM_QUIT or WM_CLOSE or 0)
            {
                return;
            }
        }
    }

    void CreateD2D()
    {
        RECT windowRect;
        GetClientRect(hWnd, &windowRect);
        clientSize = new SizeI(windowRect.right - windowRect.left, windowRect.bottom - windowRect.top);

        // 开始创建工厂创建 D3D 的逻辑
        var dxgiFactory2 = DXGI.DXGI.CreateDXGIFactory1<DXGI.IDXGIFactory2>();

        var hardwareAdapter = GetHardwareAdapter(dxgiFactory2)
            // 这里 ToList 只是想列出所有的 IDXGIAdapter1 方便调试而已。在实际代码里，大部分都是获取第一个
            .ToList().FirstOrDefault();

        if (hardwareAdapter == null)
        {
            throw new InvalidOperationException("Cannot detect D3D11 adapter");
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

        // 转换完成，可以减少对 ID3D11Device1 的引用计数
        // 调用 Dispose 不会释放掉刚才申请的 D3D 资源，只是减少引用计数
        d3D11Device.Dispose();
        d3D11DeviceContext.Dispose();

        // 创建设备，接下来就是关联窗口和交换链
        DXGI.Format colorFormat = DXGI.Format.B8G8R8A8_UNorm;

        const int FrameCount = 2;

        DXGI.SwapChainDescription1 swapChainDescription = new()
        {
            Width = clientSize.Width,
            Height = clientSize.Height,
            Format = colorFormat,
            BufferCount = FrameCount,
            BufferUsage = DXGI.Usage.RenderTargetOutput,
            SampleDescription = DXGI.SampleDescription.Default,
            Scaling = DXGI.Scaling.Stretch,
            SwapEffect = DXGI.SwapEffect.FlipDiscard,
            AlphaMode = AlphaMode.Ignore,
        };

        // 设置是否全屏
        DXGI.SwapChainFullscreenDescription fullscreenDescription = new DXGI.SwapChainFullscreenDescription
        {
            Windowed = true
        };

        // 给创建出来的窗口挂上交换链
        DXGI.IDXGISwapChain1 swapChain = 
            dxgiFactory2.CreateSwapChainForHwnd(d3D11Device1, hWnd, swapChainDescription, fullscreenDescription);

        // 不要被按下 alt+enter 进入全屏
        dxgiFactory2.MakeWindowAssociation(hWnd, DXGI.WindowAssociationFlags.IgnoreAltEnter);

        D3D11.ID3D11Texture2D backBufferTexture = swapChain.GetBuffer<D3D11.ID3D11Texture2D>(0);

        // 获取到 dxgi 的平面，这个平面就约等于窗口渲染内容
        DXGI.IDXGISurface dxgiSurface = backBufferTexture.QueryInterface<DXGI.IDXGISurface>();

        // 对接 D2D 需要创建工厂
        D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();

        // 方法1：
        //var renderTargetProperties = new D2D.RenderTargetProperties(PixelFormat.Premultiplied);

        //// 在窗口的 dxgi 的平面上创建 D2D 的画布，如此即可让 D2D 绘制到窗口上
        //D2D.ID2D1RenderTarget d2D1RenderTarget =
        //    d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);
        //var renderTarget = d2D1RenderTarget;

        // 方法2：
        // 创建 D2D 设备，通过设置 ID2D1DeviceContext 的 Target 输出为 dxgiSurface 从而让 ID2D1DeviceContext 渲染内容渲染到窗口上
        // 如 https://learn.microsoft.com/en-us/windows/win32/direct2d/images/devicecontextdiagram.png 图
        // 获取 DXGI 设备，用来创建 D2D 设备
        DXGI.IDXGIDevice dxgiDevice = d3D11Device1.QueryInterface<DXGI.IDXGIDevice>();
        D2D.ID2D1Device d2dDevice = d2DFactory.CreateDevice(dxgiDevice);
        D2D.ID2D1DeviceContext d2dDeviceContext = d2dDevice.CreateDeviceContext();

        D2D.ID2D1Bitmap1 d2dBitmap = d2dDeviceContext.CreateBitmapFromDxgiSurface(dxgiSurface);
        d2dDeviceContext.Target = d2dBitmap;

        var renderTarget = d2dDeviceContext;

        // 开启后台渲染线程，无限刷新

        var stopwatch = Stopwatch.StartNew();
        var count = 0;

        Task.Factory.StartNew(() =>
        {
            var ellipseInfoList = new List<DrawingInfo>();
            for (int i = 0; i < 100; i++)
            {
                // 随意创建颜色
                var color = new Color4((byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255), (byte) Random.Shared.Next(255));
                D2D.ID2D1SolidColorBrush brush = renderTarget.CreateSolidColorBrush(color);
                ellipseInfoList.Add(new DrawingInfo(new System.Numerics.Vector2(Random.Shared.Next(clientSize.Width), Random.Shared.Next(clientSize.Height)), new Size(Random.Shared.Next(10, 100)), brush));
            }
            while (true)
            {
                // 开始绘制逻辑
                renderTarget.BeginDraw();

                // 清空画布
                renderTarget.Clear(new Color4(0xFF, 0xFF, 0xFF));

                // 在下面绘制漂亮的界面

                for (var i = 0; i < ellipseInfoList.Count; i++)
                {
                    var drawingInfo = ellipseInfoList[i];
                    var vector2 = drawingInfo.Offset;
                    vector2.X += Random.Shared.Next(200) - 100;
                    vector2.Y += Random.Shared.Next(200) - 100;

                    while (vector2.X < 100 || vector2.X > clientSize.Width - 100)
                    {
                        vector2.X = Random.Shared.Next(clientSize.Width);
                    }

                    while (vector2.Y < 100 || vector2.Y > clientSize.Height - 100)
                    {
                        vector2.Y = Random.Shared.Next(clientSize.Height);
                    }

                    ellipseInfoList[i] = drawingInfo with { Offset = vector2 };
                    renderTarget.FillEllipse(new D2D.Ellipse(vector2, drawingInfo.Size.Width, drawingInfo.Size.Height), drawingInfo.Brush);
                }

                renderTarget.EndDraw();

                swapChain.Present(1, DXGI.PresentFlags.None);
                // 等待刷新
                d3D11DeviceContext1.Flush();

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
    }
}

static IEnumerable<DXGI.IDXGIAdapter1> GetHardwareAdapter(DXGI.IDXGIFactory2 factory)
{
    DXGI.IDXGIFactory6? factory6 = factory.QueryInterfaceOrNull<DXGI.IDXGIFactory6>();
    if (factory6 != null)
    {
        // 先告诉系统，要高性能的显卡
        for (int adapterIndex = 0;
             factory6.EnumAdapterByGpuPreference(adapterIndex, DXGI.GpuPreference.HighPerformance,
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

            //factory6.Dispose();

            Console.WriteLine($"枚举到 {adapter.Description1.Description} 显卡");
            yield return adapter;
        }

        factory6.Dispose();
    }

    // 如果枚举不到，那系统返回啥都可以
    for (int adapterIndex = 0;
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

static LRESULT WndProc(HWND hWnd, uint message, WPARAM wParam, LPARAM lParam)
{
    return DefWindowProc(hWnd, message, wParam, lParam);
}

readonly record struct DrawingInfo(System.Numerics.Vector2 Offset, Size Size, D2D.ID2D1SolidColorBrush Brush);
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/66f9fe05baba8ad30495069aebd447b160484215/CedageawhakairnerewhalNaibiferenagifee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/66f9fe05baba8ad30495069aebd447b160484215/CedageawhakairnerewhalNaibiferenagifee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 66f9fe05baba8ad30495069aebd447b160484215
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 66f9fe05baba8ad30495069aebd447b160484215
```

获取代码之后，进入 CedageawhakairnerewhalNaibiferenagifee 文件夹

更多关于 DirectX 和 D2D 相关技术请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

交流 Vortice 技术，欢迎加群： 622808968




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# Vortice 使用 DirectComposition 显示透明窗口

通过 DirectComposition 配合 WS_EX_LAYERED 或 WS_EX_NOREDIRECTIONBITMAP 窗口样式，可以让窗口高性能地背景透明，完全依靠 DWM 将窗口背景和桌面画面合成

<!--more-->
<!-- CreateTime:2026/02/04 07:15:58 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D,渲染 -->
<!-- 置顶1 -->

本文是[渲染相关系列博客](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )中的一篇，该系列博客已按照逻辑顺序编排，方便大家依次阅读。如您对渲染相关感兴趣，可以通过以下链接访问整个系列：[渲染相关系列博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

在 [DirectX 使用 Vortice 从零开始控制台创建 Direct2D1 窗口修改颜色](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html ) 博客中和大家介绍了最简方式创建了窗口和对接了 DirectX 层。在此基础上，大家也能看到此时创建的窗口是无法应用透明背景效果的

即使强行设置 `SwapChainDescription1.AlphaMode` 为 `AlphaMode.Premultiplied` 也会在 `IDXGIFactory2.CreateSwapChainForHwnd` 报错

传统 Win32 应用可以通过 UpdateLayeredWindow 方法设置窗口透明，然而 UpdateLayeredWindow 是有比较大的性能代价的，详细请参阅 [WPF 从最底层源代码了解 AllowsTransparency 性能差的原因](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E6%9C%80%E5%BA%95%E5%B1%82%E6%BA%90%E4%BB%A3%E7%A0%81%E4%BA%86%E8%A7%A3-AllowsTransparency-%E6%80%A7%E8%83%BD%E5%B7%AE%E7%9A%84%E5%8E%9F%E5%9B%A0.html )

性能较好的透明窗口实现可参阅 [WPF 制作支持点击穿透的高性能的透明背景异形窗口](https://blog.lindexi.com/post/WPF-%E5%88%B6%E4%BD%9C%E6%94%AF%E6%8C%81%E7%82%B9%E5%87%BB%E7%A9%BF%E9%80%8F%E7%9A%84%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84%E9%80%8F%E6%98%8E%E8%83%8C%E6%99%AF%E5%BC%82%E5%BD%A2%E7%AA%97%E5%8F%A3.html )

以上是在 WPF 框架里面帮忙封装好的，现在咱只有纯控制台，需要自己手动干一些活

为了方便大家阅读，本文将重新从零控制台开始，先创建好 WS_EX_LAYERED 的窗口，再将 DirectX 对接上去。总代码控制在 500 行左右。额外，为了方便 Win32 方法调用，本文还请出了 CsWin32 库，详细使用方法请参阅 
[dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html )

## 准备工作

按照 .NET 惯例，先安装一些库。本文的 D2D 基本没有戏份，仅用于绘制一点用于辅助测试的内容，本身此技术就和 D2D 无关

```xml
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
  </ItemGroup>
```

安装之后的 csproj 项目文件代码如下

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
  </ItemGroup>

</Project>
```

如以上代码所示，本文提供的代码也是 AOT 友好的。在我的测试中，构建的 32 位的程序只需 2.12 MB 的体积。如果不知道本文的项目是如何组织的，可以在本文末尾找到本文全部代码的下载方法，拉取代码了解更多细节

添加 NativeMethods.txt 文件，添加以下内容，让 CsWin32 辅助生成一些代码

```csharp
EnumDisplayMonitors
GetMonitorInfo
MONITORINFOEXW
EnumDisplaySettings
GetDisplayConfigBufferSizes
QueryDisplayConfig
DisplayConfigGetDeviceInfo
DISPLAYCONFIG_SOURCE_DEVICE_NAME
DISPLAYCONFIG_TARGET_DEVICE_NAME

RegisterClassEx
GetModuleHandle
LoadCursor
IDC_ARROW
WndProc
CreateWindowEx
CW_USEDEFAULT
ShowWindow
SW_SHOW
GetMessage
TranslateMessage
DispatchMessage
DefWindowProc
GetClientRect
WM
WM_PAINT
GetWindowLong
SetWindowLong
DwmIsCompositionEnabled
UpdateLayeredWindow
DwmExtendFrameIntoClientArea
DCompositionCreateDevice
```

以上提供的列表是超过本文所用范围的，多了也没有什么关系，一来这是测试项目，二来发布的时候 AOT 带裁剪

## 创建窗口

创建窗口的步骤和 [上一篇博客](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html ) 提供的方法十分接近，只是需要配置 WS_EX_LAYERED 样式，核心代码如下

```csharp
        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_OVERLAPPEDWINDOW
                                  | WINDOW_EX_STYLE.WS_EX_LAYERED; // Layered 是透明窗口的关键（但即使没有设置也没关系）
        var style = WNDCLASS_STYLES.CS_OWNDC | WNDCLASS_STYLES.CS_HREDRAW | WNDCLASS_STYLES.CS_VREDRAW;

        var defaultCursor = LoadCursor(
            new HINSTANCE(IntPtr.Zero), new PCWSTR(IDC_ARROW.Value));

        var className = $"lindexi-{Guid.NewGuid().ToString()}";
        var title = "The Title";
        fixed (char* pClassName = className)
        fixed (char* pTitle = title)
        {
            var wndClassEx = new WNDCLASSEXW
            {
                cbSize = (uint) Marshal.SizeOf<WNDCLASSEXW>(),
                style = style,
                lpfnWndProc = new WNDPROC(WndProc),
                hInstance = new HINSTANCE(GetModuleHandle(null).DangerousGetHandle()),
                hCursor = defaultCursor,
                hbrBackground = new HBRUSH(IntPtr.Zero),
                lpszClassName = new PCWSTR(pClassName)
            };
            ushort atom = RegisterClassEx(in wndClassEx);

            var dwStyle = WINDOW_STYLE.WS_OVERLAPPEDWINDOW;

            var windowHwnd = CreateWindowEx(
                exStyle,
                new PCWSTR((char*) atom),
                new PCWSTR(pTitle),
                dwStyle,
                0, 0, 1900, 1000,
                HWND.Null, HMENU.Null, HINSTANCE.Null, null);

            return windowHwnd;
        }
```

以上代码放在 CreateWindow 方法中。在开始之前，也先调用 DwmIsCompositionEnabled 方法，判断是否可用

```csharp
        DwmIsCompositionEnabled(out var compositionEnabled);

        if (!compositionEnabled)
        {
            Console.WriteLine($"无法启用透明窗口效果");
        }
```

预期在 Win10 以上系统都是能使用的，除非系统被魔改

窗口的消息处理代码 WndProc 先不着急写，等待完成渲染部分的逻辑再一起写

完成窗口创建之后，即可将窗口显示出来，代码如下

```csharp
        var window = CreateWindow();
        ShowWindow(window, SHOW_WINDOW_CMD.SW_NORMAL);
```

随后先开启独立的线程作为渲染线程，再跑起来消息循环

渲染线程相关逻辑，我封装到 RenderManager 类型里面，其代码如下

```csharp
        var renderManager = new RenderManager(window);
        renderManager.StartRenderThread();
```

跑起来渲染线程之后，使用标准的消息循环跑起来应用

```csharp
        while (true)
        {
            var msg = new MSG();
            var getMessageResult = GetMessage(&msg, HWND, 0,
                0);

            if (!getMessageResult)
            {
                break;
            }

            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
```

在 RenderManager 里面也提供窗口尺寸变更的方法，可以在消息循环中调用。此时的消息循环的核心代码如下

```csharp
    private LRESULT WndProc(HWND hwnd, uint message, WPARAM wParam, LPARAM lParam)
    {
        switch ((WindowsMessage)message)
        {
            case WindowsMessage.WM_NCCALCSIZE:
            {
                return new LRESULT(0);
            }
            case WindowsMessage.WM_SIZE:
            {
                RenderManager?.ReSize();
                break;
            }
        }

        return DefWindowProc(hwnd, message, wParam, lParam);
    }
```

以上的 WM_NCCALCSIZE 用于声明客户区，通过直接返回 0 告诉系统整个区域都是客户区。否则将出现标题栏不可见，但是点击有效，能拖动窗口也能在原本的最小化、最大化、关闭窗口按钮所在的位置点击响应对应的功能

透明窗口的实现在窗口创建过程中，最关键点只有 WS_EX_LAYERED 和 WM_NCCALCSIZE 的逻辑 ~~。其中 WM_NCCALCSIZE 最为关键。不带 WS_EX_LAYERED 只会出现边框而已~~

以上为采用 WS_EX_LAYERED 的方式，此方法没有采用 WS_EX_NOREDIRECTIONBITMAP 来的高效。如 [Windows 窗口样式 什么是 WS_EX_NOREDIRECTIONBITMAP 样式](https://blog.lindexi.com/post/Windows-%E7%AA%97%E5%8F%A3%E6%A0%B7%E5%BC%8F-%E4%BB%80%E4%B9%88%E6%98%AF-WS_EX_NOREDIRECTIONBITMAP-%E6%A0%B7%E5%BC%8F.html ) 博客所述，可以知道，对比采用 WS_EX_LAYERED 分层窗口的方式需要由 CPU 处理分层窗口导致的性能损耗（不一定，详见注），采用 WS_EX_NOREDIRECTIONBITMAP 方式配合 DirectComposition 可以实现更加高性能的窗口透明效果，直接将 DirectComposition 的像素交给 DWM 合成，可让全过程发生在 GPU 中，无 CPU-GPU 的拷贝损耗和带宽占用

> 注： 从 Windows 8 和 8.1 开始，虽然 User32 一直没有发生显著变化，但细微变化也有，即完全支持 GPU 上的每像素 alpha 值混合处理，且将窗口表面传输到系统内存的费用也取消了。也就是说，如果我不需要执行每像素命中测试，现在就能够生成分层窗口效果，同时不会对性能造成影响
> https://learn.microsoft.com/zh-cn/archive/msdn-magazine/2014/june/windows-with-c-high-performance-window-layering-using-the-windows-composition-engine

由于本文在编写的时候，本金鱼忘记了 WS_EX_NOREDIRECTIONBITMAP 配置，后面想加上时，担心破坏文章的连贯性，于是将 WS_EX_NOREDIRECTIONBITMAP 部分安排在本文末尾的追加内容中

## 渲染线程

独立的渲染线程也是 WPF 等 UI 框架所采用的方式，只需要新建一个线程跑起来就可以了，如果有心的话，再设置线程为 STA 的就更好，代码如下

```csharp
unsafe class RenderManager(HWND hwnd)
{
    public HWND HWND => hwnd;
    private readonly Format _colorFormat = Format.B8G8R8A8_UNorm;

    public void StartRenderThread()
    {
        var thread = new Thread(() => { RenderCore(); })
        {
            IsBackground = true,
            Name = "Render"
        };
        thread.Priority = ThreadPriority.Highest;
        thread.Start();
    }
}
```

以上的 RenderCore 就是核心的渲染方法了

由于渲染线程是独立的，不能在 ReSize 方法直接修改渲染线程相关的逻辑。本文这里简单使用一个字段表示窗口尺寸变更，需要渲染线程修改交换链尺寸

```csharp
unsafe class RenderManager(HWND hwnd)
{
    public void ReSize()
    {
        _isReSize = true;
    }

    private bool _isReSize;
}
```

在 RenderCore 的一开始就是执行初始化逻辑，初始化为本文的关键，核心就是对接 DirectComposition 实现透明窗口效果

## 初始化渲染

先获取客户区，即窗口尺寸，此尺寸用于后续交换链的创建

```csharp
        RECT windowRect;
        GetClientRect(HWND, &windowRect);
        var clientSize = new SizeI(windowRect.right - windowRect.left, windowRect.bottom - windowRect.top);
```

按照  [上一篇博客](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html ) 提供的方法获取显卡信息，代码如下

```csharp
        var dxgiFactory2 = DXGI.CreateDXGIFactory1<IDXGIFactory2>();

        IDXGIAdapter1? hardwareAdapter = GetHardwareAdapter(dxgiFactory2)
            // 这里 ToList 只是想列出所有的 IDXGIAdapter1 在实际代码里，大部分都是获取第一个
            .ToList().FirstOrDefault();
        if (hardwareAdapter == null)
        {
            throw new InvalidOperationException("Cannot detect D3D11 adapter");
        }

    private static IEnumerable<IDXGIAdapter1> GetHardwareAdapter(IDXGIFactory2 factory)
    {
        using IDXGIFactory6? factory6 = factory.QueryInterfaceOrNull<IDXGIFactory6>();
        if (factory6 != null)
        {
            // 这个系统的 DX 支持 IDXGIFactory6 类型
            // 先告诉系统，要高性能的显卡
            for (uint adapterIndex = 0;
                 factory6.EnumAdapterByGpuPreference(adapterIndex, GpuPreference.HighPerformance,
                     out IDXGIAdapter1? adapter).Success;
                 adapterIndex++)
            {
                if (adapter == null)
                {
                    continue;
                }

                AdapterDescription1 desc = adapter.Description1;
                if ((desc.Flags & AdapterFlags.Software) != AdapterFlags.None)
                {
                    // Don't select the Basic Render Driver adapter.
                    adapter.Dispose();
                    continue;
                }

                Console.WriteLine($"枚举到 {adapter.Description1.Description} 显卡");
                yield return adapter;
            }
        }
        else
        {
            // 不支持就不支持咯，用旧版本的方式获取显示适配器接口
        }

        // 如果枚举不到，那系统返回啥都可以
        for (uint adapterIndex = 0;
             factory.EnumAdapters1(adapterIndex, out IDXGIAdapter1? adapter).Success;
             adapterIndex++)
        {
            AdapterDescription1 desc = adapter.Description1;

            if ((desc.Flags & AdapterFlags.Software) != AdapterFlags.None)
            {
                // Don't select the Basic Render Driver adapter.
                adapter.Dispose();

                continue;
            }

            Console.WriteLine($"枚举到 {adapter.Description1.Description} 显卡");
            yield return adapter;
        }
    }
```

尝试创建 ID3D11Device 设备，代码如下

```csharp
        FeatureLevel[] featureLevels = new[]
        {
            FeatureLevel.Level_12_2,
            FeatureLevel.Level_12_1,
            FeatureLevel.Level_12_0,
            FeatureLevel.Level_11_1,
            FeatureLevel.Level_11_0,
            FeatureLevel.Level_10_1,
            FeatureLevel.Level_10_0,
            FeatureLevel.Level_9_3,
            FeatureLevel.Level_9_2,
            FeatureLevel.Level_9_1,
        };

        IDXGIAdapter1 adapter = hardwareAdapter;
        DeviceCreationFlags creationFlags = DeviceCreationFlags.BgraSupport;
        var result = D3D11.D3D11CreateDevice
        (
            adapter,
            DriverType.Unknown,
            creationFlags,
            featureLevels,
            out ID3D11Device d3D11Device, out FeatureLevel featureLevel,
            out ID3D11DeviceContext d3D11DeviceContext
        );
```

本文以上的 `FeatureLevel[]` 中添加了 Level_12_2 等的不合理需求，如果创建失败了，则执行降级逻辑。按照技术原理，只需有 Level_11_1 即可

```csharp
        if (result.Failure)
        {
            // 降低等级试试
            featureLevels = new[]
            {
                //FeatureLevel.Level_12_2,
                //FeatureLevel.Level_12_1,
                //FeatureLevel.Level_12_0,
                FeatureLevel.Level_11_1,
                FeatureLevel.Level_11_0,
                FeatureLevel.Level_10_1,
                FeatureLevel.Level_10_0,
                FeatureLevel.Level_9_3,
                FeatureLevel.Level_9_2,
                FeatureLevel.Level_9_1,
            };

            result = D3D11.D3D11CreateDevice
            (
                adapter,
                DriverType.Unknown,
                creationFlags,
                featureLevels,
                out d3D11Device, out featureLevel,
                out d3D11DeviceContext
            );
        }
```

将获取到的 ID3D11Device 当成 ID3D11Device1 设备来用，这一步基本不会遇到出错的，代码如下

```csharp
        // 大部分情况下，用的是 ID3D11Device1 和 ID3D11DeviceContext1 类型
        // 从 ID3D11Device 转换为 ID3D11Device1 类型
        ID3D11Device1 d3D11Device1 = d3D11Device.QueryInterface<ID3D11Device1>();
        var d3D11DeviceContext1 = d3D11DeviceContext.QueryInterface<ID3D11DeviceContext1>();

        // 获取到了新的两个接口，就可以减少 `d3D11Device` 和 `d3D11DeviceContext` 的引用计数。调用 Dispose 不会释放掉刚才申请的 D3D 资源，只是减少引用计数
        d3D11Device.Dispose();
        d3D11DeviceContext.Dispose();
```

准备交换链参数，代码如下

```csharp
        // 缓存的数量，包括前缓存。大部分应用来说，至少需要两个缓存，这个玩过游戏的伙伴都知道
        const int FrameCount = 2;
        SwapChainDescription1 swapChainDescription = new()
        {
            Width = (uint) clientSize.Width,
            Height = (uint) clientSize.Height,
            Format = _colorFormat,
            BufferCount = FrameCount,
            BufferUsage = Usage.RenderTargetOutput,
            SampleDescription = SampleDescription.Default,
            Scaling = Scaling.Stretch,
            SwapEffect = SwapEffect.FlipSequential, // 使用 FlipSequential 配合 Composition
            AlphaMode = AlphaMode.Premultiplied,
            Flags = SwapChainFlags.None,
        };
```

交换链中有以下参数必须固定为此搭配：

- Scaling： Scaling.Stretch
- SwapEffect： SwapEffect.FlipDiscard 或 SwapEffect.FlipSequential ，正常来说都会采用 FlipSequential 配合 Composition 使用
- AlphaMode： AlphaMode.Premultiplied 。使用 `AlphaMode.Ignore` 和 `AlphaMode.Unspecified` 参数也是合法的，但是如此就丢失了窗口透明了，不是咱的需求。而 `AlphaMode.Straight` 参数则是不搭的

如果以上参数不搭配，则会在创建交换链时，返回 0x887A0001 错误

上文提到了 `AlphaMode.Premultiplied` 预乘，简单来说就是最终输出的值里的 RGB 分量都乘以透明度。更多细节请参阅 [支持的像素格式和 Alpha 模式 - Win32 apps - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/direct2d/supported-pixel-formats-and-alpha-modes#about-premultiplied-and-straight-alpha-modes )

判断系统版本，决定能否使用 DirectComposition 功能，代码如下

```csharp
        // 使用 DirectComposition 才能支持透明窗口
        bool useDirectComposition = true;
        // 使用 DirectComposition 时有系统版本要求
        useDirectComposition = useDirectComposition & OperatingSystem.IsWindowsVersionAtLeast(8, 1);
```

如此可以让代码走两个分支，使用 DirectComposition 的分支的代码如下

```csharp
        IDXGISwapChain1 swapChain;

        if (useDirectComposition)
        {
            // 使用 CreateSwapChainForComposition 创建支持预乘 Alpha 的 SwapChain
            swapChain =
                dxgiFactory2.CreateSwapChainForComposition(d3D11Device1, swapChainDescription);

            // 创建 DirectComposition 设备和目标
            IDXGIDevice dxgiDevice = d3D11Device1.QueryInterface<IDXGIDevice>();
            IDCompositionDevice compositionDevice = DComp.DCompositionCreateDevice<IDCompositionDevice>(dxgiDevice);
            compositionDevice.CreateTargetForHwnd(HWND, true, out IDCompositionTarget compositionTarget);

            // 创建视觉对象并设置 SwapChain 作为内容
            IDCompositionVisual compositionVisual = compositionDevice.CreateVisual();
            compositionVisual.SetContent(swapChain);
            compositionTarget.SetRoot(compositionVisual);
            compositionDevice.Commit();
        }
```

从上面代码可见核心步骤是先让 CreateSwapChainForComposition 创建出交换链对象。再将 ID3D11Device1 当成 IDXGIDevice 设备，用于调用 DComp.DCompositionCreateDevice 创建出 IDCompositionDevice 设备

调用 IDCompositionDevice 的 CreateTargetForHwnd 方法即可为当前的窗口挂上 IDCompositionTarget 对象。随后再调用 IDCompositionDevice 设备的 CreateVisual 创建 IDCompositionVisual 视觉对象。将刚才创建出来的交换链作为视觉对象的内容，如此即可完成交换链与内容的绑定

```csharp
            // 创建视觉对象并设置 SwapChain 作为内容
            IDCompositionVisual compositionVisual = compositionDevice.CreateVisual();
            compositionVisual.SetContent(swapChain);
```

现在交换链所渲染的画面已经能够到 IDCompositionVisual 里了，再将 IDCompositionVisual 作为 IDCompositionTarget 的根，即可让 IDCompositionVisual 参与 DWM 合成

```csharp
            compositionTarget.SetRoot(compositionVisual);
            compositionDevice.Commit();
```

如果没有 DirectComposition 可用，则依然使用上一篇博客介绍的方法创建交换链，代码如下

```csharp
        IDXGISwapChain1 swapChain;

        if (useDirectComposition)
        {
            ...
        }
        else
        {
            var fullscreenDescription = new SwapChainFullscreenDescription()
            {
                Windowed = true,
            };

            swapChainDescription.AlphaMode = AlphaMode.Ignore;

            swapChain = dxgiFactory2.CreateSwapChainForHwnd(d3D11Device1, hwnd, swapChainDescription,
                fullscreenDescription);
        }
```

以上代码的 DirectComposition 为本文的核心，只需要创建出输出带预乘的交换链，配合 WS_EX_LAYERED 窗口，即可渲染出透明窗口

接下来的逻辑就是和 D2D 对接，尝试渲染透明的界面用于测试

## 对接渲染

由于 D2D 没有什么戏份，本文就只贴出核心代码

```csharp
        using D2D.ID2D1Factory1 d2DFactory = D2D.D2D1.D2D1CreateFactory<D2D.ID2D1Factory1>();

                var d3D11Texture2D = _renderContext.SwapChain.GetBuffer<ID3D11Texture2D>(0);

                var dxgiSurface = d3D11Texture2D.QueryInterface<IDXGISurface>();
                var renderTargetProperties = new D2D.RenderTargetProperties()
                {
                    PixelFormat = new PixelFormat(D2DColorFormat, Vortice.DCommon.AlphaMode.Premultiplied),
                    Type = D2D.RenderTargetType.Hardware,
                };

                D2D.ID2D1RenderTarget d2D1RenderTarget =
                    d2DFactory.CreateDxgiSurfaceRenderTarget(dxgiSurface, renderTargetProperties);

        while (!_isDisposed)
        {
                D2D.ID2D1RenderTarget renderTarget = d2D1RenderTarget;

                renderTarget.BeginDraw();

                var color = new Color4(Random.Shared.NextSingle(), Random.Shared.NextSingle(),
                    Random.Shared.NextSingle(), 0.1f);
                renderTarget.Clear(color);

                renderTarget.EndDraw();

                _renderContext.SwapChain.Present(1, PresentFlags.None);
                _renderContext.D3D11DeviceContext1.Flush();
        }
```

如果准备处理窗口尺寸改变，则需要在循环里面判断 `_isReSize` 字段，调用交换链的 ResizeBuffers 方法，代码如下

```csharp
            if (_isReSize)
            {
                // 处理窗口大小变化
                _isReSize = false;

                GetClientRect(HWND, out var pClientRect);
                var clientSize = new SizeI(pClientRect.right - pClientRect.left, pClientRect.bottom - pClientRect.top);

                var swapChain = _renderContext.SwapChain;

                swapChain.ResizeBuffers(2,
                    (uint) (clientSize.Width),
                    (uint) (clientSize.Height),
                    _colorFormat,
                    SwapChainFlags.None
                );
            }
```

尝试运行代码，可见一个不断闪烁的背景透明的窗口

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/369de6b65c4122cec6a6c9ffbcc0b352a419e83e/DirectX/D2D/FarjairyakaBurnefuwache) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/369de6b65c4122cec6a6c9ffbcc0b352a419e83e/DirectX/D2D/FarjairyakaBurnefuwache) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 369de6b65c4122cec6a6c9ffbcc0b352a419e83e
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 369de6b65c4122cec6a6c9ffbcc0b352a419e83e
```

获取代码之后，进入 DirectX/D2D/FarjairyakaBurnefuwache 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

## 采用 WS_EX_NOREDIRECTIONBITMAP 的方案

上文提及的是采用 WS_EX_LAYERED 分层窗口的方式，此方式与采用 WS_EX_NOREDIRECTIONBITMAP 的方案的代码写法非常接近，只是 WS_EX_NOREDIRECTIONBITMAP 可以带来更高的性能

其改动全在 CreateWindow 中，只需将 WINDOW_EX_STYLE 从 WS_EX_LAYERED 换成 WS_EX_NOREDIRECTIONBITMAP 即可

在使用 WS_EX_NOREDIRECTIONBITMAP 中，还可以在 WndProc 干掉 WM_NCCALCSIZE 用来保留窗口边框，即窗口标题栏

修改之后的 CreateWindow 方法中的 WINDOW_EX_STYLE 配置如下

```csharp
    private unsafe HWND CreateWindow()
    {
        DwmIsCompositionEnabled(out var compositionEnabled);

        if (!compositionEnabled)
        {
            Console.WriteLine($"无法启用透明窗口效果");
        }

        // [Windows 窗口样式 什么是 WS_EX_NOREDIRECTIONBITMAP 样式](https://blog.lindexi.com/post/Windows-%E7%AA%97%E5%8F%A3%E6%A0%B7%E5%BC%8F-%E4%BB%80%E4%B9%88%E6%98%AF-WS_EX_NOREDIRECTIONBITMAP-%E6%A0%B7%E5%BC%8F.html )
        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;

        var style = WNDCLASS_STYLES.CS_OWNDC | WNDCLASS_STYLES.CS_HREDRAW | WNDCLASS_STYLES.CS_VREDRAW;

        var defaultCursor = LoadCursor(
            new HINSTANCE(IntPtr.Zero), new PCWSTR(IDC_ARROW.Value));

        var className = $"lindexi-{Guid.NewGuid().ToString()}";
        var title = "The Title";
        fixed (char* pClassName = className)
        fixed (char* pTitle = title)
        {
            var wndClassEx = new WNDCLASSEXW
            {
                cbSize = (uint) Marshal.SizeOf<WNDCLASSEXW>(),
                style = style,
                lpfnWndProc = new WNDPROC(WndProc),
                hInstance = new HINSTANCE(GetModuleHandle(null).DangerousGetHandle()),
                hCursor = defaultCursor,
                hbrBackground = new HBRUSH(IntPtr.Zero),
                lpszClassName = new PCWSTR(pClassName)
            };
            ushort atom = RegisterClassEx(in wndClassEx);

            var dwStyle = WINDOW_STYLE.WS_OVERLAPPEDWINDOW | WINDOW_STYLE.WS_VISIBLE;

            var windowHwnd = CreateWindowEx(
                exStyle,
                new PCWSTR((char*) atom),
                new PCWSTR(pTitle),
                dwStyle,
                0, 0, 1900, 1000,
                HWND.Null, HMENU.Null, HINSTANCE.Null, null);

            return windowHwnd;
        }
    }
```

可见核心关键改动只有 `WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;` 这一行

如果期望显示标题栏，则可以继续在 WndProc 进行改动，删除 WM_NCCALCSIZE 相关代码，删除之后的代码如下

```csharp
    private LRESULT WndProc(HWND hwnd, uint message, WPARAM wParam, LPARAM lParam)
    {
        if ((WindowsMessage) message == WindowsMessage.WM_SIZE)
        {
            _renderManager?.ReSize();
        }

        return DefWindowProc(hwnd, message, wParam, lParam);
    }
```

更改使用 WS_EX_NOREDIRECTIONBITMAP 的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/5b79f1c45819750fa9e931d78b3797a81c877294/DirectX/D2D/NearajurkeekallnoYabarfoge) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5b79f1c45819750fa9e931d78b3797a81c877294/DirectX/D2D/NearajurkeekallnoYabarfoge) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5b79f1c45819750fa9e931d78b3797a81c877294
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5b79f1c45819750fa9e931d78b3797a81c877294
```

获取代码之后，进入 DirectX/D2D/NearajurkeekallnoYabarfoge 文件夹，即可获取到源代码

## 性能提醒

无论是采用 WS_EX_LAYERED 还是 WS_EX_NOREDIRECTIONBITMAP 的方案，都会导致 DWM 需要更多的性能损耗

如果自己的应用是没有透明背景需求的，建议不要带上这两个选项以降低 DWM 合成的损耗。或依然保持窗口样式，但在交换链配置 `swapChainDescription.AlphaMode = AlphaMode.Ignore;` 也能减少 DWM 的损耗

只有在 4k 全屏应用里面，才能比较明显看到开启透明窗口带来的性能损耗

### 性能测试

为了说明性能的差异，我做了对比测试

本次实验的机器配置如下：

- 屏幕： 3840x2160 (4K) + 百分百 DPI
- CPU： i5-12450H
- GPU： 集显

测试结果如下：

#### DirectCompositionPremultipliedAlphaMode

采用 DirectComposition 和配置交换链的 AlphaMode 为 Premultiplied 预乘，此为本文的演示透明窗口组合效果。代码片段如下

```csharp
        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;

        SwapChainDescription1 swapChainDescription = new()
        {
            Width = (uint) clientSize.Width,
            Height = (uint) clientSize.Height,
            Format = _colorFormat,
            BufferCount = FrameCount,
            BufferUsage = Usage.RenderTargetOutput,
            SampleDescription = SampleDescription.Default,
            Scaling = Scaling.Stretch,
            SwapEffect = SwapEffect.FlipSequential, // 使用 FlipSequential 配合 Composition
            AlphaMode = AlphaMode.Premultiplied,
            Flags = SwapChainFlags.None,
        };

        var swapChain = dxgiFactory2.CreateSwapChainForComposition(d3D11Device1, swapChainDescription);
```

运行效果：

- CPU： 忽略不计
- GPU： 占用 75-80
- DWM： 占用 65-70
- 满帧率

可见此时会让 DWM 非常繁忙。整体的 GPU 占用比较高

#### DirectCompositionWithWS_EX_LAYERED

和 DirectCompositionPremultipliedAlphaMode 不同的是，仅将窗口样式从 WS_EX_NOREDIRECTIONBITMAP 换成 WS_EX_LAYERED 而已，代码差异如下

```diff
-       WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;
+       WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_LAYERED;

        SwapChainDescription1 swapChainDescription = new()
        {
            Width = (uint) clientSize.Width,
            Height = (uint) clientSize.Height,
            Format = _colorFormat,
            BufferCount = FrameCount,
            BufferUsage = Usage.RenderTargetOutput,
            SampleDescription = SampleDescription.Default,
            Scaling = Scaling.Stretch,
            SwapEffect = SwapEffect.FlipSequential, // 使用 FlipSequential 配合 Composition
            AlphaMode = AlphaMode.Premultiplied,
            Flags = SwapChainFlags.None,
        };

        var swapChain = dxgiFactory2.CreateSwapChainForComposition(d3D11Device1, swapChainDescription);
```

运行效果：

- CPU： 忽略不计
- GPU： 占用 77-85
- DWM： 占用 75-80
- 满帧率

可见采用 WS_EX_LAYERED 样式基本和 WS_EX_NOREDIRECTIONBITMAP 持平，差异几乎测试不到，符合文档说明内容

#### DirectCompositionIgnoreAlphaMode

和 DirectCompositionPremultipliedAlphaMode 不同的是，仅将 AlphaMode 设置为忽略，此时将丢失窗口透明效果

```diff
        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;

-       swapChainDescription.AlphaMode = AlphaMode.Premultiplied;
+       swapChainDescription.AlphaMode = AlphaMode.Ignore;

        var swapChain = dxgiFactory2.CreateSwapChainForComposition(d3D11Device1, swapChainDescription);
```

运行效果：

- CPU： 忽略不计
- GPU： 占用 5
- DWM： 0-1
- 满帧率

此时的窗口没有透明效果，聪明的 DWM 也就没有需要执行合成逻辑，自然占用也就少了

#### DirectCompositionWithoutWS_EX_NOREDIRECTIONBITMAP

和 DirectCompositionPremultipliedAlphaMode 不同的是，去掉了窗口的 WS_EX_NOREDIRECTIONBITMAP 样式，此时也会丢失窗口的透明效果，且这个组合是不正确的，正常不会有人会这么做

```diff
-        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;
+        WINDOW_EX_STYLE exStyle = default;

        swapChainDescription.AlphaMode = AlphaMode.Premultiplied;

        var swapChain = dxgiFactory2.CreateSwapChainForComposition(d3D11Device1, swapChainDescription);
```

运行效果：

- CPU： 忽略不计
- GPU： 占用 45-50
- DWM： 40
- 满帧率

这个组合比较奇怪，跑出来的效果也比较奇怪。我感觉是因为此时需要重定向表面，导致存在一些带宽损耗

#### CreateSwapChainForHwnd

直接使用 CreateSwapChainForHwnd 传统方式，不再使用 DirectComposition 创建，丢失窗口透明效果，代码差异如下

```diff
        WINDOW_EX_STYLE exStyle = WINDOW_EX_STYLE.WS_EX_NOREDIRECTIONBITMAP;

-       swapChainDescription.AlphaMode = AlphaMode.Premultiplied;
+       swapChainDescription.AlphaMode = AlphaMode.Ignore;

-       var swapChain = dxgiFactory2.CreateSwapChainForComposition(d3D11Device1, swapChainDescription);
+       var swapChain = dxgiFactory2.CreateSwapChainForHwnd(d3D11Device1, HWND, swapChainDescription, fullscreenDescription);        
```

运行效果：

- CPU： 忽略不计
- GPU： 占用 5
- DWM： 0-1
- 满帧率

此时可见使用传统的 CreateSwapChainForHwnd 方式和 DirectComposition 忽略 AlphaMode 的性能持平，此时两者都能达到较好的效果

无论是对 CreateSwapChainForHwnd 设置的 WINDOW_EX_STYLE 为无或 WS_EX_LAYERED 都对此结果没有影响

### 性能测试代码

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b678a0cc6d689aa63fd1239bb88113ff8a4b9fcd/DirectX/D2D/LurqificelgallRikurneawekearner) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b678a0cc6d689aa63fd1239bb88113ff8a4b9fcd/DirectX/D2D/LurqificelgallRikurneawekearner) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b678a0cc6d689aa63fd1239bb88113ff8a4b9fcd
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b678a0cc6d689aa63fd1239bb88113ff8a4b9fcd
```

获取代码之后，进入 DirectX/D2D/LurqificelgallRikurneawekearner 文件夹，即可获取到源代码

## 更多博客

渲染部分，关于 SharpDx 和 Vortice 的使用方法，包括入门级教程，请参阅：

- [渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
- [SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
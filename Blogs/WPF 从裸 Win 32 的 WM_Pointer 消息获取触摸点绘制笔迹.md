本文将告诉大家如何在 WPF 里面，接收裸 Win 32 的 WM_Pointer 消息，从消息里面获取触摸点信息，使用触摸点信息绘制简单的笔迹

<!--more-->


<!-- CreateTime:2024/09/01 07:15:43 -->
<!-- 置顶2 -->
<!-- 发布 -->
<!-- 博客 -->

开始之前必须说明的是使用本文的方法不会带来什么优势，既不能带来笔迹书写上的加速，也不能带来笔迹效果的平滑，且代码复杂。本文唯一的作用只是让大家了解一下基础机制

需要再次说明的是，在 WPF 里面，开启了 WM_Pointer 消息之后，通过 Touch 或 Stylus 事件收到的信息也是从 WM_Pointer 消息里面过来的。大家可以尝试在 Touch 事件监听函数添加断点，通过堆栈可以看到是从 Windows 消息循环来的

可以从调用堆栈看到如下函数，此函数就是核心的 WPF 框架里面从 WM_Pointer 消息获取触摸信息的代码

```
>	PresentationCore.dll!System.Windows.Interop.HwndPointerInputProvider.System.Windows.Interop.IStylusInputProvider.FilterMessage(nint hwnd, MS.Internal.Interop.WindowMessage msg, nint wParam, nint lParam, ref bool handled)
```

这个 FilterMessage 函数的大概代码如下

```csharp
	nint IStylusInputProvider.FilterMessage(nint hwnd, WindowMessage msg, nint wParam, nint lParam, ref bool handled)
	{
		handled = false;
		if (PointerLogic.IsEnabled)
		{
			switch (msg)
			{
			case WindowMessage.WM_ENABLE:
				IsWindowEnabled = MS.Win32.NativeMethods.IntPtrToInt32(wParam) == 1;
				break;
			case WindowMessage.WM_POINTERENTER:
				handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.InRange, Environment.TickCount);
				break;
			case WindowMessage.WM_POINTERUPDATE:
				handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.Move, Environment.TickCount);
				break;
			case WindowMessage.WM_POINTERDOWN:
				handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.Down, Environment.TickCount);
				break;
			case WindowMessage.WM_POINTERUP:
				handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.Up, Environment.TickCount);
				break;
			case WindowMessage.WM_POINTERLEAVE:
				handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.OutOfRange, Environment.TickCount);
				break;
			}
		}
		return IntPtr.Zero;
	}
```

由此可以了解到，使用本文自己从 Win32 消息获取的触摸信息，和从 WPF 提供的 Touch 或 Stylus 事件里面获取的触摸信息的来源是相同的

这时候也许有人会说，在 WPF 里面经过了一些封装，可能性能不如自己写的。我只想说，不要过于自信了哦。且别忘了消息是从 UI 线程里面获取的，无论你用不用 WPF 的事件，在 WPF 底层的解析消息获取触摸数据引发事件的代码都会跑，也就是无论你用不用，需要 WPF 干的活一点都没少。只有一个 UI 线程的情况下，如果用自己解析的，那还会多一点点处理逻辑，完全不如直接使用 WPF 的。再加上 WPF 的解析部分没有多少代码，如果有做性能分析的话，可以看到甚至做路由事件时的命中测试，判断命中到哪个控件和引发事件等逻辑的耗时远比解析来的多。且解析消息的数据耗时接近无法被直接测量出来，即测量所需时间大于解析的性能

科普就到这里，如果对 WPF 触摸相关感兴趣，请看 [WPF 触摸相关](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

为了能够在消息里面收到 POINTER 消息，我根据 [WPF dotnet core 如何开启 Pointer 消息的支持](https://blog.lindexi.com/post/WPF-dotnet-core-%E5%A6%82%E4%BD%95%E5%BC%80%E5%90%AF-Pointer-%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html ) 博客提供的方法，在 App 构造函数里面添加如下代码开启 Pointer 消息的支持。本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

```csharp
    public App()
    {
        AppContext.SetSwitch("Switch.System.Windows.Input.Stylus.EnablePointerSupport", true);
    }
```

接下来按照 [WPF 如何确定应用程序开启了 Pointer 触摸消息的支持](https://blog.lindexi.com/post/WPF-%E5%A6%82%E4%BD%95%E7%A1%AE%E5%AE%9A%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%90%AF%E4%BA%86-Pointer-%E8%A7%A6%E6%91%B8%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html ) 博客提供的方法添加消息监听处理逻辑，如以下代码

```csharp
    public MainWindow()
    {
        InitializeComponent();

        SourceInitialized += OnSourceInitialized;
    }

    private void OnSourceInitialized(object? sender, EventArgs e)
    {
        var windowInteropHelper = new WindowInteropHelper(this);
        var hwnd = windowInteropHelper.Handle;

        HwndSource source = HwndSource.FromHwnd(hwnd)!;
        source.AddHook(Hook);
    }

    private unsafe IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {
        ... // 忽略其他代码
        return IntPtr.Zero;
    }
```

再定义上一些消息常量，然后跑起来代码确定 Pointer 消息开启成功

```csharp
    private unsafe IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {
        const int WM_POINTERDOWN = 0x0246;
        const int WM_POINTERUPDATE = 0x0245;
        const int WM_POINTERUP = 0x0247;

        if (msg is WM_POINTERDOWN or WM_POINTERUPDATE or WM_POINTERUP)
        {
             // 在这里打断点，如果能进断点则证明 Pointer 消息开启成功
        }

        ... // 忽略其他代码
        return IntPtr.Zero;
    }
```

以下逻辑需要调用一些 Win32 的 API 函数，为了方便使用，根据 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html ) 博客提供的方法，使用 CsWin32 库简化 Win32 函数调用逻辑，可以减少大量的 PInvoke 定义

可以避免定义错 PInvoke 函数导致的诡异失败

编辑 csproj 项目文件，替换为如下代码用于快速安装 CsWin32 库

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Windows.CsWin32" PrivateAssets="all" Version="0.3.106" />
  </ItemGroup>
</Project>
```

大家可以看到以上的项目文件代码的 OutputType 被我设置为 exe 类型，如此启动项目将会有默认的控制台，方便我在控制台输出内容

按照 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html ) 博客提供的方法添加 `NativeMethods.txt` 文件，在此文件里面添加一些代码需要用到的 Win32 函数

```
GetPointerTouchInfo
ScreenToClient
RegisterTouchWindow
WM_TOUCH
GetTouchInputInfo
GetPointerDeviceRects
ClientToScreen
```

在 `NativeMethods.txt` 文件添加的是所需的 Win32 函数名，添加之后将会由 CsWin32 库使用源代码生成器方式生成对应的 PInvoke 代码和参数所需的类型，如结构体和枚举

根据 WPF 的源代码，先将消息过来的 wparam 转换为 `pointerId` 参数，代码如下

```csharp
            var pointerId = (uint) (ToInt32(wparam) & 0xFFFF);
            PInvoke.GetPointerTouchInfo(pointerId, out var info);
```

这里需要额外说明的是这个 `pointerId` 参数不等于设备 Id 号，即如 WPF 的 `TouchDevice.Id` 等，这是不相同的，需要使用 [GetPointerCursorId](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointercursorid) 进行关联才能拿到和 WPF 一样的值。但是使用 `pointerId` 参数去区分不同的触摸点还是可以的

如此即可拿到核心的 [POINTER_INFO](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_info) 结构体对象

```csharp
            POINTER_INFO pointerInfo = info.pointerInfo;
```

简单处理的话，拿到的 `pointerInfo` 的 `ptPixelLocation` 字段就是当前触摸的坐标点了，采用的是像素坐标，使用屏幕坐标系

```csharp
            var point = pointerInfo.ptPixelLocation;
```

从屏幕坐标系转换为 WPF 坐标系，代码如下

```csharp
            PInvoke.ScreenToClient(new HWND(hwnd), ref point);
```

不考虑 DPI 的情况下，这样就可以使用了

按照 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html) 博客提供的方法进行笔迹对接即可绘制出笔迹

这就是最简单的从 Win32 消息接收 Pointer 消息绘制笔迹的方法

然而以上的方法也存在不少的问题，比如忽略了 DPI 问题，以及精度问题。在大尺寸触摸屏上，直接使用 `ptPixelLocation` 字段将会画出锯齿的笔迹。如下图，黑色的线是直接使用 `ptPixelLocation` 字段收到的触摸点连接的折线

<!-- ![](image/WPF 记一个特别简单的点集滤波平滑方法/WPF 记一个特别简单的点集滤波平滑方法2.png) -->
![](http://cdn.lindexi.site/lindexi%2F2024830160364905.jpg)

上图红色的曲线是使用 [WPF 记一个特别简单的点集滤波平滑方法](https://blog.lindexi.com/post/WPF-%E8%AE%B0%E4%B8%80%E4%B8%AA%E7%89%B9%E5%88%AB%E7%AE%80%E5%8D%95%E7%9A%84%E7%82%B9%E9%9B%86%E6%BB%A4%E6%B3%A2%E5%B9%B3%E6%BB%91%E6%96%B9%E6%B3%95.html ) 博客提供的方法进行平滑的笔迹线
<!-- [WPF 记一个特别简单的点集滤波平滑方法 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18387840 ) -->

在大屏触摸设备上，从硬件层面就有一层平滑算法了，但是受限于硬件的计算资源，只有简单的平滑。在 Windows 的 WISPTIS 模块里面，也会对触摸做一定的平滑算法，如丢弃某些过于离谱的触摸点。关于 Windows 上的 WISPTIS 模块的平滑算法属于我和系统软件，即软硬件工程师，进行合作测试出来的，他输入的点和我使用 BusHound 抓到得点和 WPF 层报告的点做对比，可以看到硬件层发送过来的点和 BusHound 抓到的相同，而和 WPF 层报告的点大部分情况下相同，只有某些点被丢弃。被丢弃的点是我这边设计的杂点。但是如果报告的触摸点，有瞬间飞到 0,0 点的情况，那这个 0,0 点则不会被丢弃

在 WPF 层上，从消息到 Touch 事件这里，是不会对点的坐标进行处理，不会执行平滑算法，最多只有做控件坐标转换。在 WPF 的 Ink 模块里面才会对输入的点做更进一步的平滑处理

我对比了从 Pointer 消息的 `ptPixelLocation` 字段收到的触摸点对接的 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html) 博客提供的方法，和原始博客提供的程序，可以看到还是原来的笔迹更加顺滑

其核心原因在于 Pointer 消息的 `ptPixelLocation` 字段拿到的是丢失精度的点，像素为单位。如果在精度稍微高的触摸屏下，将会有明显的锯齿效果

如果想要获取比较高精度的触摸点，可以使用 `ptHimetricLocationRaw` 字段。这里需要对后缀 Raw 作出更多的说明，在微软[官方文档](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_info)里面说了不带 Raw 的是预测的值，即 `ptPixelLocation` 是预测的像素坐标点，而 `ptPixelLocationRaw` 是不带预测的像素坐标点。对于咱如果是使用在笔迹上，其实更应该使用的是 `ptPixelLocationRaw` 是不带预测的像素坐标点。否则预测效果可能会导致毛刺

使用 `ptHimetricLocationRaw` 字段会稍微复杂，由于 ptHimetricLocationRaw 采用的是 pointerDeviceRect 坐标系，需要转换到屏幕坐标系

转换方法就是先将 `ptHimetricLocationRaw` 的 X 坐标，压缩到 `[0-1]` 范围内，然后乘以 `displayRect` 的宽度，再加上 `displayRect` 的 left 值，即得到了屏幕坐标系的 X 坐标。压缩到 `[0-1]` 范围内的方法就是除以 `pointerDeviceRect` 的宽度。同理可以计算 Y 坐标

以上的 `displayRect` 和 `pointerDeviceRect` 需要使用 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 函数获取

```csharp
            global::Windows.Win32.Foundation.RECT pointerDeviceRect = default;
            global::Windows.Win32.Foundation.RECT displayRect = default;

            PInvoke.GetPointerDeviceRects(pointerInfo.sourceDevice, &pointerDeviceRect, &displayRect);
```

以上代码用到了不安全代码，记得给 Hook 函数标记上 unsafe 作为不安全代码

根据上文提供的算法，编写如下代码将 `ptHimetricLocationRaw` 转换为 WPF 坐标系的点

```csharp
            // 如果想要获取比较高精度的触摸点，可以使用 ptHimetricLocationRaw 字段
            // 由于 ptHimetricLocationRaw 采用的是 pointerDeviceRect 坐标系，需要转换到屏幕坐标系
            // 转换方法就是先将 ptHimetricLocationRaw 的 X 坐标，压缩到 [0-1] 范围内，然后乘以 displayRect 的宽度，再加上 displayRect 的 left 值，即得到了屏幕坐标系的 X 坐标。压缩到 [0-1] 范围内的方法就是除以 pointerDeviceRect 的宽度
            // 为什么需要加上 displayRect.left 的值？考虑多屏的情况，屏幕可能是副屏
            // Y 坐标同理
           var point2D = new Point2D(
                pointerInfo.ptHimetricLocationRaw.X / (double) pointerDeviceRect.Width * displayRect.Width +
                displayRect.left,
                pointerInfo.ptHimetricLocationRaw.Y / (double) pointerDeviceRect.Height * displayRect.Height +
                displayRect.top);
```

以上代码的 Point2D 类型的定义如下

```csharp
readonly record struct Point2D(double X, double Y);
```

以上代码获取的是屏幕坐标系的点，需要转换到 WPF 坐标系

转换过程的两个重点：

1.底层 ClientToScreen 只支持整数类型，直接转换会丢失精度。即使是 WPF 封装的 PointFromScreen 或 PointToScreen 方法也会丢失精度

2.需要进行 DPI 换算，必须要求 DPI 感知

先测量窗口与屏幕的偏移量，这里直接取 0 0 点即可，因为这里获取到的是虚拟屏幕坐标系，不需要考虑多屏的情况

```csharp
            var screenTranslate = new Point(0, 0);
            PInvoke.ClientToScreen(new HWND(hwnd), ref screenTranslate);
```

获取当前的 DPI 值

```csharp
            var dpi = VisualTreeHelper.GetDpi(this);
```

先做平移，再做 DPI 换算

```csharp
            point2D = new Point2D(point2D.X - screenTranslate.X, point2D.Y - screenTranslate.Y);
            point2D = new Point2D(point2D.X / dpi.DpiScaleX, point2D.Y / dpi.DpiScaleY);
```

此时拿到的 point2D 就是 WPF 坐标系的点了，但是拿这个点对接笔迹，如以下代码

```csharp
            if (msg == WM_POINTERUPDATE)
            {
                var strokeVisual = GetStrokeVisual(pointerId);
                strokeVisual.Add(new StylusPoint(point2D.X, point2D.Y));
                strokeVisual.Redraw();
            }
            else if (msg == WM_POINTERUP)
            {
                StrokeVisualList.Remove(pointerId);
            }
```

运行代码即可看到可以在较高精度触摸屏上绘制出比较顺滑的笔迹

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/322313ee55d0eeaae7148b24ca279e1df087871e/WPFDemo/DefilireceHowemdalaqu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/322313ee55d0eeaae7148b24ca279e1df087871e/WPFDemo/DefilireceHowemdalaqu) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 322313ee55d0eeaae7148b24ca279e1df087871e
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 322313ee55d0eeaae7148b24ca279e1df087871e
```

获取代码之后，进入 WPFDemo/DefilireceHowemdalaqu 文件夹，即可获取到源代码

更多 WPF 触摸相关技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

# WPF 通过 GetRawPointerDeviceData 从 WM_POINTER 消息触摸裸数据

本文将告诉大家如何在 WPF 里面，通过 GetRawPointerDeviceData 方法从 WM_POINTER 指针消息触摸裸数据，以及解析数据信息获取到更多触摸框上报的信息

<!--more-->
<!-- CreateTime:2025/05/22 07:08:22 -->

<!-- 发布 -->
<!-- 博客 -->

正常来说，在 WPF 里面开启 WM_POINTER 指针消息即可在框架层获取到触摸裸消息。详细请看 [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%BB%8E-WM_POINTER-%E6%B6%88%E6%81%AF%E5%88%B0-Touch-%E4%BA%8B%E4%BB%B6.html )
<!-- [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18403860 ) -->

本文的内容实际是对 [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%BB%8E-WM_POINTER-%E6%B6%88%E6%81%AF%E5%88%B0-Touch-%E4%BA%8B%E4%BB%B6.html ) 博客的补充，说明从 GetRawPointerDeviceData 获取 WM_POINTER 消息触摸裸数据的细节，避免这部分细节内容过多影响到上一篇博客的内容

本文所涉及的技术不仅可以在 WPF 上使用，同样也适用在 Avalonia 或 UNO 或 CPF 等框架上。只是当前 Avalonia 依然没有所用最裸的方式获取触摸消息，从而导致 Avalonia 的触摸消息没有 WPF 的全，就看后面我能否吵架吵得赢，去改改 Avalonia 底层触摸了，让 Avalonia 也走和 WPF 一样的触摸方式

本文属于触摸的深水区，阅读本文，你将知道如何从最底层的方式获取到 WM_POINTER 上报的最裸和最全的未经加工投毒的触摸信息。如对 WPF 触摸和笔迹感兴趣，还请参阅 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )

开始之前，先回顾一下如何在 WPF 里开启 Pointer 消息的支持：只需在 App 构造函数添加如下代码即可，详细请参阅 [WPF dotnet core 如何开启 Pointer 消息的支持](https://blog.lindexi.com/post/WPF-dotnet-core-%E5%A6%82%E4%BD%95%E5%BC%80%E5%90%AF-Pointer-%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html )

```csharp
    public App()
    {
        AppContext.SetSwitch("Switch.System.Windows.Input.Stylus.EnablePointerSupport", true);
    }
```

为了方便演示，本文将采用 .NET 9 版本的 WPF 框架，如大家想要在低版本的 WPF 进行测试，还请自行处理好兼容性问题。预计正常来说是不会遇到的，除非直接降级到 .NET Framework 版本

在完成在 WPF 配置开启 Pointer 消息支持之后，按照 [WPF 如何确定应用程序开启了 Pointer 触摸消息的支持](https://blog.lindexi.com/post/WPF-%E5%A6%82%E4%BD%95%E7%A1%AE%E5%AE%9A%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%90%AF%E4%BA%86-Pointer-%E8%A7%A6%E6%91%B8%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html) 博客提供的方法进行测试能否收到 WM_Pointer 消息。于此同时也能建立好 Hook 钩子

```csharp
        public MainWindow()
        {
            InitializeComponent();

            SourceInitialized += OnSourceInitialized;
        }

        private void OnSourceInitialized(object sender, EventArgs e)
        {
            var windowInteropHelper = new WindowInteropHelper(this);
            var hwnd = windowInteropHelper.Handle;

            HwndSource source = HwndSource.FromHwnd(hwnd);
            source.AddHook(Hook);
        }

        private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            const int WM_POINTERDOWN = 0x0246;

            if (msg == WM_POINTERDOWN)
            {
                // 开启了 Pointer 消息
                Debugger.Break();
            }


            return IntPtr.Zero;
        }
```

由于本文将需要用到几个 Win32 函数，但我不想自己去定义，于是就用了微软提供的 CsWin32 库来减少工作量，详细请看 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html )

具体做法就是按照 .NET 惯例安装上 CsWin32 库，安装之后的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
    <IncludeNativeLibrariesForSelfExtract>true</IncludeNativeLibrariesForSelfExtract>
    <EnableCompressionInSingleFile>true</EnableCompressionInSingleFile>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Windows.CsWin32" Version="0.3.183">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

在项目里面放入 NativeMethods.txt 文件，添加本文将用到或没用到的几个 Win32 方法或定义

```
SetWindowsHookEx
CallNextHookEx
GetWindowLong
SetWindowLong
SetWindowLongPtr
SetWindowLongPtrW
CallWindowProc
PostMessage
WM_POINTE*
GetPointerDeviceProperties
GetRawPointerDeviceData
GetPointerTouchInfo
GetPointerDeviceRects
GetPointerDevices
GetPointerCursorId
```

如果大家看到这里不知道如何搭建项目也没关系，我在本文末尾放了本文所有代码的下载方法，拉取本文的代码自己跑跑看就知道项目如何搭建

修改一下 MainWindow.xaml 文件，添加一点不好看的界面。界面里面只有一个 TouchInfoTextBlock 控件和一个 TouchSizeBorder 控件。其中 TouchInfoTextBlock 用来显示触摸的数据，而 TouchSizeBorder 用来显示触摸的尺寸\面积\宽高

```xml
    <Grid>
        <TextBlock x:Name="TouchInfoTextBlock" IsHitTestVisible="False" HorizontalAlignment="Center" VerticalAlignment="Center" FontSize="15"></TextBlock>
        <Border x:Name="TouchSizeBorder" Visibility="Collapsed" IsHitTestVisible="False" BorderThickness="2" BorderBrush="Gray" HorizontalAlignment="Left" VerticalAlignment="Top">
            <Border.RenderTransform>
                <TranslateTransform></TranslateTransform>
            </Border.RenderTransform>
        </Border>
    </Grid>
```

准备工作完成了，接下来的核心代码就全在 Hook 里面了

先删掉对 WM_POINTERDOWN 的测试代码，将其换成 WM_POINTERUPDATE 消息，在移动的过程中获取触摸消息才能看到不断的刷屏，特别有感觉。同时也能照顾一些红外触摸框，在落下的一瞬获取到的触摸面积是不准确的问题

在 CsWin32 库里面，会给很多方法都标记上 SupportedOSPlatform 声明系统版本，这一点非常棒，防止不小心在低版本系统进入了只有高版本才有提供的方法的分支。但在本文示例项目里面，能进入分支的，必然是收到了 Pointer 消息。能够收到 Pointer 消息，必然就是 Win8 或以上的系统版本了，也基本上也可以认为是 Win10 系统也好了

```csharp
    private unsafe IntPtr Hook(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
    {
        if (msg == WM_POINTERUPDATE /*Pointer Update*/)
        {
            Debug.Assert(OperatingSystem.IsWindowsVersionAtLeast(10, 0), "能够收到 WM_Pointer 消息，必定系统版本号不会低");
        }

        return 0;
    }
```

在收到 WM_POINTERUPDATE 消息时，按照微软和咱的约定，消息里的 wParam 就是 PointerId 号，转换代码如下

```csharp
    private unsafe IntPtr Hook(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
    {
        if (msg == WM_POINTERUPDATE /*Pointer Update*/)
        {
            Debug.Assert(OperatingSystem.IsWindowsVersionAtLeast(10, 0), "能够收到 WM_Pointer 消息，必定系统版本号不会低");

            var pointerId = (uint) (ToInt32(wParam) & 0xFFFF);
           
        }

        return 0;
    }

    private static int ToInt32(WPARAM wParam) => ToInt32((IntPtr) wParam.Value);
    private static int ToInt32(IntPtr ptr) => IntPtr.Size == 4 ? ptr.ToInt32() : (int) (ptr.ToInt64() & 0xffffffff);
```

拿到 PointerId 号后，即可调用 GetPointerTouchInfo 方法获取具体的触摸信息。这里需要额外说明的是 PointerId 号是一个系统层虚拟的概念，和在 WPF 里面获取到的触摸设备的 Id 是不相同的，在 WPF 里面获取到的触摸设备 Id 是和 CursorId 游标 Id 相关联的，细节请参阅 [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%BB%8E-WM_POINTER-%E6%B6%88%E6%81%AF%E5%88%B0-Touch-%E4%BA%8B%E4%BB%B6.html )
<!-- [dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18403860 ) -->

从 GetPointerTouchInfo 拿到的触摸消息，在很多情况下就已经足够用了，但如果要追求获取更多触摸上报的信息的，就需要继续通过 GetRawPointerDeviceData 获取更多数据了。从 GetPointerTouchInfo 方法获取到的就是通过 [指针输入消息和通知传递](https://learn.microsoft.com/zh-cn/windows/win32/inputmsg/messages-and-notifications-portal) 的标准属性，按照本文提供的方式获取则可以获取超过指针（WM_Pointer）消息的标准属性数量的更多属性

在使用 GetRawPointerDeviceData 之前，需要先准备好一些基础信息，比如屏幕尺寸信息，当前的触摸设备将会上报的信息有哪些等

获取 Pointer 下的屏幕信息是调用 [GetPointerDeviceRects](https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 方法，代码如下

```csharp
            global::Windows.Win32.Foundation.RECT pointerDeviceRect = default;
            global::Windows.Win32.Foundation.RECT displayRect = default;

            GetPointerDeviceRects(pointerInfo.sourceDevice, &pointerDeviceRect, &displayRect);
```

这里面会拿到两个值，第一个是 Pointer 设备上报的范围有多大，另一个就是显示屏的范围是多少。是否有伙伴好奇，为什么给定的是范围而不是尺寸呢？别忘了多屏的存在哦，非主屏的显示屏的范围的左上角可以是负数坐标的哦

拿到这两个值之后，就可以在后续获取到裸触摸信息的坐标时，通过 Pointer 设备上报的范围的比值，求出对应在显示屏幕上的坐标，再叠加 DPI 缩放和窗口坐标系偏移，即可计算出 Pointer 坐标点到窗口坐标系的触摸点坐标之间的关系

继续获取当前的触摸设备将会上报的信息有哪些，基本来说最简单的触摸屏也会至少应该要上报 X Y 坐标点，复杂一点的触摸屏会多上报更多信息，比如触摸的宽度、高度信息，触摸旋转角、触摸压感等等信息。这些信息是在触摸屏的 HID 描述符信息里面就带上的，咱可以通过 [GetPointerDeviceProperties](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties) 获取到和 Pointer 关联的信息。经过我拿了几个触摸框进行测试对比，发现从 [GetPointerDeviceProperties](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties) 拿到的信息和触摸框本身 HID 描述符上报的信息是一致的，证明这个过程里面没有在系统层进行加工。不像是 Linux X11 里面会自作聪明将触摸的宽度高度进行转换成 TouchMajor 和 TouchMinor 信息

```csharp
            uint propertyCount = 0;
            GetPointerDeviceProperties(pointerInfo.sourceDevice, &propertyCount, null);
            POINTER_DEVICE_PROPERTY* pointerDevicePropertyArray =
                stackalloc POINTER_DEVICE_PROPERTY[(int) propertyCount];
            GetPointerDeviceProperties(pointerInfo.sourceDevice, &propertyCount, pointerDevicePropertyArray);
```

获取当前的触摸设备将会上报的信息有哪些时，调用 [GetPointerDeviceProperties](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties) 两次的原因是第一次只是用来获取数量，获取到数量就用 C# 的 stackalloc 在栈上直接申请内存，毕竟这个信息的数据量都是很少的，用不要钱的栈内存速度会更快

在本文示例项目里面，都是在每次 WM_POINTERUPDATE 触摸移动消息过来的时候获取各种信息的，我原本以为这样做可能会影响到性能。然而实际测试来说，这部分耗时接近可以忽略。由于这种触摸、系统相关的 API 调用耗时不好进行测量，我就没有做基准测试。如果大家担心这部分性能问题的话，还请自行测试一下，但必须说明的是我的测试结果是认为这部分耗时接近可以忽略，性能测量误差可能会很大

通过 [GetPointerDeviceProperties](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties) 方法即可获取到当前的触摸设备将会上报的信息，每个属性信息就是一个 POINTER_DEVICE_PROPERTY 结构体，如 X 和 Y 两个轴的坐标信息，就分别会是两个 POINTER_DEVICE_PROPERTY 结构体对象信息

打开 [POINTER_DEVICE_PROPERTY](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/ns-winuser-pointer_device_property) 结构体看看其定义，代码如下

```csharp
		/// <summary>Contains pointer-based device properties (Human Interface Device (HID) global items that correspond to HID usages).</summary>
		/// <remarks>Developers can use this function to determine the properties that a device supports beyond the standard ones that are delivered through <a href="https://docs.microsoft.com/windows/win32/inputmsg/messages-and-notifications-portal">Pointer Input Messages and Notifications</a>. The properties map directly to HID usages.</remarks>
		[global::System.CodeDom.Compiler.GeneratedCode("Microsoft.Windows.CsWin32", "0.3.183+73e6125f79.RR")]
		internal partial struct POINTER_DEVICE_PROPERTY
		{
			/// <summary>The minimum value that the device can report for this property.</summary>
			internal int logicalMin;

			/// <summary>The maximum value that the device can report for this property.</summary>
			internal int logicalMax;

			/// <summary>The physical minimum  in Himetric.</summary>
			internal int physicalMin;

			/// <summary>The physical maximum in Himetric.</summary>
			internal int physicalMax;

			/// <summary>The unit.</summary>
			internal uint unit;

			/// <summary>The exponent.</summary>
			internal uint unitExponent;

			/// <summary>The usage page for the property, as documented in the HID specification.</summary>
			internal ushort usagePageId;

			/// <summary>The usage  of  the property, as documented in the HID specification.</summary>
			internal ushort usageId;
		}
```

熟悉 HID 的伙伴也许一下就看出来了，这个结构体的各个属性的定义和 HID 描述符的定义信息是相同的，都有逻辑与物理的最大和最小值，都有带单位等信息。正如[官方文档](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/ns-winuser-pointer_device_property)所述，属性直接映射到 HID 用法

根据 HID 标准规范，从 Usage PageId 和 UsageId 就可以知道这是一个什么属性的信息了。如 `UsagePageId == 0x01 && UsageId == 0x30` 就表示 X 轴的信息，如 `UsagePageId == 0x01 && UsageId == 0x31` 就表示 Y 轴信息。这部分属于 HID 规范了，详细规范请参阅 <https://www.usb.org/document-library/hid-usage-tables-16>

常用的定义枚举信息如下，以下代码是从 WPF 仓库抄的

```csharp
/// <summary>
///
/// WM_POINTER stack must parse out HID spec usage pages
/// <see cref="http://www.usb.org/developers/hidpage/Hut1_12v2.pdf"/>
/// </summary>
/// Copy from https://github.com/dotnet/wpf
internal enum HidUsagePage : ushort
{
    Undefined = 0x00,
    Generic = 0x01,
    Simulation = 0x02,
    Vr = 0x03,
    Sport = 0x04,
    Game = 0x05,
    Keyboard = 0x07,
    Led = 0x08,
    Button = 0x09,
    Ordinal = 0x0a,
    Telephony = 0x0b,
    Consumer = 0x0c,
    Digitizer = 0x0d,
    Unicode = 0x10,
    Alphanumeric = 0x14,
    BarcodeScanner = 0x8C,
    WeighingDevice = 0x8D,
    MagneticStripeReader = 0x8E,
    CameraControl = 0x90,
    MicrosoftBluetoothHandsfree = 0xfff3,
}

/// <summary>
///
/// 
/// WISP pre-parsed these, WM_POINTER stack must do it itself
/// 
/// See Stylus\biblio.txt - 1
/// <see cref="http://www.usb.org/developers/hidpage/Hut1_12v2.pdf"/> 
/// </summary>
/// Copy from https://github.com/dotnet/wpf
internal enum HidUsage
{
    X = 0x30,
    Y = 0x31,
    Z = 0x32,
    TipPressure = 0x30,
    BarrelPressure = 0x31,
    XTilt = 0x3D,
    YTilt = 0x3E,
    Azimuth = 0x3F,
    Altitude = 0x40,
    Twist = 0x41,
    TipSwitch = 0x42,
    SecondaryTipSwitch = 0x43,
    BarrelSwitch = 0x44,
    TouchConfidence = 0x47,
    Width = 0x48,
    Height = 0x49,
    TransducerSerialNumber = 0x5B,
}

enum DigitizersUsageId : ushort
{
    Width = 0x48,
    Height = 0x49,
    ContactIdentifier = 0x51
}
```

依据 Usage PageId 和 UsageId 信息，将获取到的触摸属性信息进行分类，其代码如下

```csharp
            var xPropertyIndex = -1;
            var yPropertyIndex = -1;
            var contactIdentifierPropertyIndex = -1;
            var widthPropertyIndex = -1;
            var heightPropertyIndex = -1;

            for (var i = 0; i < pointerDevicePropertySpan.Length; i++)
            {
                POINTER_DEVICE_PROPERTY pointerDeviceProperty = pointerDevicePropertySpan[i];
                var usagePageId = pointerDeviceProperty.usagePageId;
                var usageId = pointerDeviceProperty.usageId;
                // 单位
                var unit = pointerDeviceProperty.unit;
                // 单位指数。 它与 Unit 字段一起定义了设备报告中数据的物理单位。具体来说：
                // - Unit：定义了数据的基本单位，例如厘米、英寸、弧度等。
                // - UnitExponent：表示单位的数量级（即 10 的幂次）。它用于缩放单位值，使其适应不同的范围
                var unitExponent = pointerDeviceProperty.unitExponent;
                if (usagePageId == (ushort) HidUsagePage.Generic)
                {
                    if (usageId == (ushort) HidUsage.X)
                    {
                        xPropertyIndex = i;
                    }
                    else if (usageId == (ushort) HidUsage.Y)
                    {
                        yPropertyIndex = i;
                    }
                }
                else if (usagePageId == (ushort) HidUsagePage.Digitizer)
                {
                    if (usageId == (ushort) DigitizersUsageId.Width)
                    {
                        widthPropertyIndex = i;
                    }
                    else if (usageId == (ushort) DigitizersUsageId.Height)
                    {
                        heightPropertyIndex = i;
                    }
                    else if (usageId == (ushort) DigitizersUsageId.ContactIdentifier)
                    {
                        contactIdentifierPropertyIndex = i;
                    }
                }
            }
```

如以上代码所述，我对多个属性的分类只是依靠 Index 序号而已，这样也方便我判断属性是否存在

在进行 [GetRawPointerDeviceData](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getrawpointerdevicedata) 方法调用之前，先看一下 [GetRawPointerDeviceData](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getrawpointerdevicedata) 方法的签名

```c++
BOOL GetRawPointerDeviceData
(
  [in]  UINT32                  pointerId,
  [in]  UINT32                  historyCount,
  [in]  UINT32                  propertiesCount,
  [in]  POINTER_DEVICE_PROPERTY *pProperties,
  [out] LONG                    *pValues
);
```

- pointerId: 不必多说，这就是从 wParam 传过来的参数
- historyCount: 历史点数量
- propertiesCount: 单个点包含的属性数量
- pProperties: 属性集合
- pValues: 输出读取到的点的信息

按照参数的顺序，咱一个个介绍其传入的值。历史点的数量可以从 [POINTER_INFO](https://learn.microsoft.com/windows/win32/api/winuser/ns-winuser-pointer_info) 结构体的 historyCount 获取到数值。历史点的存在的意义是当消息循环无法及时处理指针消息时，将会在系统层缓存历史点信息。历史点的更新是在下一次 WM_Pointer 消息循环时才进行的，这就意味着在相同一次的消息循环里面，多次获取历史点信息，每次都能拿到相同的值

再继续唠嗑一下历史点的存在，指针消息走的是标准 Win32 消息循环，如果每个指针消息（如触摸消息）进来的时候，都需要执行一次消息循环获取，那么将可能存在积压问题。这是什么意思呢？假定我拿到的是 VID=1FF7 厂商的高精度触摸框产生的触摸消息，触摸框上报可高达 144Hz 的频率，然而由于 UI 线程业务的存在，让我无法在消息循环里面以如此高的速度访达，这就导致了生产者-消费者不平等问题，导致消息积压。消息积压将会导致当前时刻处理的都是过去比较久远历史时刻的触摸数据，进而表现行为就是触摸延迟性

通过历史点的机制，可以让业务层根据自身的能力，成批地获取触摸点。如此可以减少处理次数。打个比方，如果业务层实在顶不住那么多的数据量，则可以根据丢点算法只取少量的点满足业务需求。丢点算法是什么呢？最简单的丢点算法就是只返回最后一个点，其他都丢掉。举一个业务上的例子来告诉大家，假定当前的业务就是用触摸拖动一个滑块。那么在这个业务情况下，真只需最后一个点，历史点的存在对此业务毫无意义，只需知道最后一个点，将滑块设置到最后一个触摸点的坐标即可完成触摸滑动。当渲染跟得上时，此时滑块将能够实现跟手的效果

单个点包含的属性数量和属性即可这两个参数都可从前面的 [GetPointerDeviceProperties](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties) 方法获取到

最后一个参数是输出读取到的点的信息，需要咱提前准备好一个数组来承接。一般来说没有开发者头铁到从栈上分配内存来承接读取到的点，这是因为历史点数量不可控，可能 UI 卡了一下，历史点数量就超级多了。准备的承接的数组长度应该就是属性数量乘以历史点数量的值，其代码如下

```csharp
            var historyCount = pointerInfo.historyCount;
            int[] rawPointerData = new int[propertyCount * historyCount];
```

以上的 `rawPointerData` 是 new 出来的，需要用 fixed 固定一下，代码如下

```csharp
            fixed (int* pValue = rawPointerData)
            {
                bool success = GetRawPointerDeviceData(pointerId, historyCount, propertyCount,
                    pointerDevicePropertyArray, pValue);
                Debug.Assert(success);
            }
```

现在获取到的数据就全都放在 `rawPointerData` 里面了，这里面都是一些 int 的数据，需要进行一些解析处理才能转换为咱人类可以理解的触摸数据

为了更进一步说明解析的逻辑，我定义了以下结构体，准备从触摸数据中解析出触摸的宽度高度信息。这里必须敲黑板，不是所有触摸框都有上报触摸的宽度高度信息，以下代码只有在上报了触摸宽度高度信息的设备上才能获取到宽度高度信息

```csharp
readonly
    record
    struct
    RawPointerPoint
    (
        int Id,
        double X,
        double Y,
        int RawWidth,
        int RawHeight,
        double PixelWidth,
        double PixelHeight,
        double PhysicalWidth,
        double PhysicalHeight
    );
```

从以上代码可以看到，我定义的 RawPointerPoint 结构体居然对于宽度高度有这么多表述方法。这也是对的哦，一个就是原始的 Pointer 指针消息上报的裸宽度高度信息，一个就是像素单位的宽度高度信息，最后一个就是物理单位的宽度高度信息

像素单位是可以自己根据 DPI 和分辨率进行计算的，按照像素单位计算之后，在触摸框给力的情况下，能够画出矩形范围完全框住触摸物体的。物理尺寸宽度高度则可以测量出实际触摸物体的物理尺寸。像素尺寸和物理尺寸是两个维度，如我有一个 96 寸的和一个 65 寸大尺寸触摸屏，这个两个触摸屏是 2k 分辨率的，我拿相同的物体在这两个屏幕上触摸。预期两个屏幕拿到的像素尺寸将会是不相同的，而物理尺寸则是相同的

完成结构体定义之后，接下来看看如何解析获取到的触摸裸数据

在 `rawPointerData` 里面，是密集按照一个个历史触摸点排列放入的数据，每个触摸点包含的数据和触摸点属性一一对应。遍历所有历史点的过程的代码如下

```csharp
            var rawPointerPoint = new RawPointerPoint();
            for (int i = 0; i < historyCount; i++)
            {
                var baseIndex = i * propertyCount;
            }
```

先来演示一下读取基础的 X 和 Y 信息，读取代码如下

```csharp
            var rawPointerPoint = new RawPointerPoint();

            for (int i = 0; i < historyCount; i++)
            {
                var baseIndex = i * propertyCount;

                if (xPropertyIndex >= 0 && yPropertyIndex >= 0)
                {
                    var xValue = rawPointerData[baseIndex + xPropertyIndex];
                    var yValue = rawPointerData[baseIndex + yPropertyIndex];
                    var xProperty = pointerDevicePropertySpan[xPropertyIndex];
                    var yProperty = pointerDevicePropertySpan[yPropertyIndex];

                    // 从 Pointer 算到的只能是屏幕坐标的点，转换进应用程序窗口坐标还需要自己再次计算
                    var xForScreen = ((double) xValue - xProperty.logicalMin) /
                        (xProperty.logicalMax - xProperty.logicalMin) * displayRect.Width;
                    var yForScreen = ((double) yValue - yProperty.logicalMin) /
                        (yProperty.logicalMax - yProperty.logicalMin) * displayRect.Height;

                    rawPointerPoint = rawPointerPoint with
                    {
                        X = xForScreen,
                        Y = yForScreen,
                    };
                }

                if (rawPointerPoint != default)
                {
                    // 默认调试只取一个点好了
                    break;
                }
            }
```

先从 `rawPointerData` 里面，按照 `xPropertyIndex` 和 `yPropertyIndex` 序号，读取出触摸裸数据信息。再获取出 X 和 Y 的描述属性，描述属性里面有逻辑的最大值和最小值，此时分别将读取到的 X 和 Y 的裸数据 `xValue` 和 `yValue` 根据逻辑最大值和最小值进行缩放，再乘以显示屏范围，即可获取到屏幕坐标系的坐标。要是卡在这一步理解的话，还请停一下，仔细思考一下

首先上报的数据都是按照 HID 规范的，值范围将在 logicalMin 和 logicalMax 之间。其次通过 `((double) xValue - xProperty.logicalMin) / (xProperty.logicalMax - xProperty.logicalMin)` 这串计算，即可计算出上报的 `xValue` 所在的 X 坐标上的比例值。最后乘以 `displayRect.Width` 屏幕像素尺寸，即可获取到当前的坐标点在 X 轴的像素点数

如上所述，当前拿到的就是 `xForScreen` 屏幕像素坐标系的，这将和 WPF 窗口坐标系之间存在一定的转换关系。转换关系有两步，第一是多屏计算逻辑，第二就是窗口左上角距离屏幕左上角之间的距离，第三就是 DPI 缩放问题。咱一个个计算开始，先是多屏计算

从 [GetPointerDeviceRects](https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 方法 拿到的 `RECT displayRect` 值，这就意味着有左上角的值。为什么存在左上角的值呢？想想如果有多个屏幕，其中当前触摸按下去的触摸屏是副屏，且是在左边的副屏。那么此时逻辑上的屏幕范围左上角就应该 X 坐标是负数。多屏计算只需叠加上 `displayRect.left` 即可

尽管 Win32 层和 WPF 层都提供了 PointToScreen 和 PointFromScreen 等类似的方法，允许从窗口坐标系和屏幕坐标系之间相互转换，然而底层 ClientToScreen 只支持整数类型，直接转换会丢失精度。即使是 WPF 封装的 PointFromScreen 或 PointToScreen 方法也会丢失精度。为了保证精度不丢失，常用做法是计算窗口 0,0 点到屏幕上的映射关系，如此减去窗口左上角即可计算屏幕坐标系到窗口坐标系的转换

最后一步是从窗口像素坐标系转换为 WPF 坐标系，转换时只需叠加上 DPI 缩放即可。以上三步的计算代码合起来如下，可以看到代码实际是非常简单的

```csharp
            // 转换为 WPF 坐标系
            var scale = VisualTreeHelper.GetDpi(this).PixelsPerDip;
            // 计算出窗口的左上角坐标对应到屏幕坐标的点
            // 为什么不是在 PointToScreen 传入坐标点，而是传入 0 点呢？这是因为经过了 PointToScreen 方法会丢失精度，即小数点之后的内容会被丢失。因此正常的计算方法都是取 0 点计算出窗口坐标系相对于屏幕坐标系的偏移量
            // 减去偏移量之后，再经过 DPI 缩放即可获取窗口坐标系的坐标
            var originPointToScreen = this.PointToScreen(new Point(0, 0));
        

            var xWpf = (rawPointerPoint.X + displayRect.left - originPointToScreen.X) / scale;
            var yWpf = (rawPointerPoint.Y + displayRect.top - originPointToScreen.Y) / scale;

            TouchInfoTextBlock.Text = $"RawPointerPoint For WPF XY={xWpf:0.00},{yWpf:0.00}";
```

这里可以尝试添加 StylusMove 事件，尝试使用 `var position = e.GetPosition(this);` 获取 WPF 事件的坐标点，对比咱以上计算的 `xWpf` 和 `yWpf` 变量，预期两者能够拿到相同的值

```csharp
    public MainWindow()
    {
        InitializeComponent();
        StylusMove += MainWindow_StylusMove;
    }

    private void MainWindow_StylusMove(object sender, StylusEventArgs e)
    {
        var position = e.GetPosition(this);
        var x = position.X;
        var y = position.Y;
        TouchInfoTextBlock.Text += $"\r\n[WPF StylusMove] Id={e.StylusDevice.Id} XY={x:0.00},{y:0.00}";
    }
```

了解了简单的 X Y 坐标的计算之后，接下来看看稍微复杂的宽度高度计算方法，核心思路和坐标点计算是相同的

同样是依靠宽度和高度属性的索引获取出原始的裸宽度高度信息，以及对应的属性值，代码如下

```csharp
            for (int i = 0; i < historyCount; i++)
            {
                var baseIndex = i * propertyCount;

                ... // 忽略其他代码

                if (widthPropertyIndex >= 0 && heightPropertyIndex >= 0)
                {
                    var widthValue = rawPointerData[baseIndex + widthPropertyIndex];
                    var heightValue = rawPointerData[baseIndex + heightPropertyIndex];

                    var widthProperty = pointerDevicePropertySpan[widthPropertyIndex];
                    var heightProperty = pointerDevicePropertySpan[heightPropertyIndex];

                    ... // 忽略其他代码
                }

                if (rawPointerPoint != default)
                {
                    // 默认调试只取一个点好了
                    break;
                }
            }
```

计算宽度高度的方法如下：

1. 计算出宽度 Value 和最大值最小值的比例
2. 按照比例计算出宽度高度在屏幕上的像素值
3. 按照比例配合物理最小值和最大值计算出宽度高度的物理值

先计算出比例，代码如下，比例就是原始值和逻辑最大最小值的比例

```csharp
var widthScale = ((double) widthValue - widthProperty.logicalMin) /
                 (widthProperty.logicalMax - widthProperty.logicalMin);

var heightScale = ((double) heightValue -
heightProperty.logicalMin) /
                  (heightProperty.logicalMax - heightProperty.logicalMin);
```

通过将宽度比例乘以 `displayRect.Width` 即可获取到像素宽度，同理可以获取到高度像素值

```csharp
var widthPixel = widthScale * displayRect.Width;
var heightPixel = heightScale * displayRect.Height;

rawPointerPoint = rawPointerPoint with
{
    RawWidth = widthValue,
    RawHeight = heightValue,
    PixelWidth = widthPixel,
    PixelHeight = heightPixel,
};
```

同样，这里获取到的是屏幕像素值的宽度高度，由于宽度高度从逻辑定义上就不存在多屏问题和窗口坐标系问题，唯一存在的就是和 WPF 坐标系之间的 DPI 转换而已。因此相对比与 X Y 坐标点的转换来说，从屏幕宽度高度像素值转换为 WPF 坐标系的宽度高度就非常简单，只需简单叠加 DPI 计算即可

```csharp
            // 转换为 WPF 坐标系
            var scale = VisualTreeHelper.GetDpi(this).PixelsPerDip;

            var widthWpf = rawPointerPoint.PixelWidth / scale;
            var heightWpf = rawPointerPoint.PixelHeight / scale;
```

在一个靠谱的触摸框上，即能够上报正确的触摸尺寸的触摸框上，咱直接拿 `widthWpf` 和 `heightWpf` 赋值到某个控件的宽度高度上，即可让这个控件的尺寸等同于触摸物体对应在屏幕的像素大小。再加上计算出来的 `xWpf` 和 `yWpf` 就能实现完全画出来框住触摸物体的效果

如以下代码，我将计算出来的 WPF 坐标系的坐标和尺寸赋值给到 TouchSizeBorder 控件，实际测试在很多触摸框上能够非常好框住触摸物体

```csharp
if (double.IsRealNumber(xWpf) && double.IsRealNumber(yWpf) && double.IsRealNumber(widthWpf) &&
    double.IsRealNumber(heightWpf))
{
    TouchSizeBorder.Visibility = Visibility.Visible;
    if (TouchSizeBorder.RenderTransform is TranslateTransform translateTransform)
    {
        translateTransform.X = xWpf - widthWpf / 2;
        translateTransform.Y = yWpf - heightWpf / 2;
    }

    TouchSizeBorder.Width = widthWpf;
    TouchSizeBorder.Height = heightWpf;
}
```

此时如果在带有正常的触摸框设备上运行程序，则可以看到 TouchSizeBorder 控件刚好能框住触摸到触摸框上的物体的外接矩形

以上为像素宽度高度尺寸的计算方法，接下来咱继续计算其物理尺寸。开始之前，为了简单起见，我这里只处理 HID 上报单位为厘米的情况，其他单位的计算方法也是相同的，大家自行决定就好。在 [POINTER_DEVICE_PROPERTY](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/ns-winuser-pointer_device_property) 结构体里面定义了 unit 和 unitExponent 两个字段，这两个字段的含义分别是单位，以及单位指数比例

什么是 unitExponent 单位指数比例？要知道，在 HID 上报上来的数据里面，是没有带浮点数单位的尺寸的，那此时如何能够表示是 0.几 的值呢？简单的方法就是设置上报的值属于 10 的 N 次方之一，这样就能很方便的指定小数点。这里的 10 的 N 次方之一中的 N 就是 unitExponent 指数比例的值，其对应关系写在了 HID hut1_6.pdf 23.18.4 Generic Unit Exponent 章里面，其关系只是一张表而已或者是一个简单的二进制关系

判断单位是否厘米，我用到了在 WPF 仓库里面的代码，代码如下

```csharp
internal static class StylusPointPropertyUnitHelper
{
    // Copy from https://github.com/dotnet/wpf

    /// <summary>
    /// Convert WM_POINTER units to WPF units
    /// </summary>
    /// <param name="pointerUnit"></param>
    /// <returns></returns>
    internal static StylusPointPropertyUnit? FromPointerUnit(uint pointerUnit)
    {
        StylusPointPropertyUnit unit = StylusPointPropertyUnit.None;

        if (_pointerUnitMap.TryGetValue(pointerUnit & UNIT_MASK, out unit))
        {
            return unit;
        }

        return (StylusPointPropertyUnit?) null;
    }

    /// <summary>
    /// Mapping for WM_POINTER based unit, taken from legacy WISP code
    /// </summary>
    private static Dictionary<uint, StylusPointPropertyUnit> _pointerUnitMap =
        new Dictionary<uint, StylusPointPropertyUnit>()
        {
            { 1, StylusPointPropertyUnit.Centimeters },
            { 2, StylusPointPropertyUnit.Radians },
            { 3, StylusPointPropertyUnit.Inches },
            { 4, StylusPointPropertyUnit.Degrees },
        };

    /// <summary>
    /// Mask to extract units from raw WM_POINTER data
    /// <see cref="http://www.usb.org/developers/hidpage/Hut1_12v2.pdf"/> 
    /// </summary>
    private const uint UNIT_MASK = 0x000F;
}
```

核心就是从 `_pointerUnitMap` 字段里面获取映射关系。当然了，如此简单的映射关系让我来写不如就写成方法内判断好了，这样速度更快，内存更省

具体判断代码如下

```csharp
if (StylusPointPropertyUnitHelper.FromPointerUnit(widthProperty.unit) ==
    StylusPointPropertyUnit.Centimeters)
{
    ... // 忽略其他代码
}
```

判断是厘米单位之后，接下来计算单位比例，如上文，单位比例通过 `unitExponent` 进行计算。其表示的是 10 的 N 次方。学过毕导的小学二年级的伙伴们都知道，十的多少次负数次方就表示10进制下的直接移动小数点多少位。如 0x0E 表示 10 的 `-2` 次方，即等于计算出的 Value 乘以 `10^-2` 等于 `Value * 0.01` 的值。我见过的触摸框里面，很大部分都选用 0x0E 的值，刚好也等于百分之一的单位的值

看到这里，伙伴们也许有疑惑，为什么采用 0x0E 表示 10 的 `-2` 次方，是存在什么样的关系让 0x0E 和 `-2` 对应起来？其实很简单，要么就是阅读 hut1_6.pdf 23.18.4 章的表格，打一张表，问就是表是这么写的。要么就是认为这只是一个简单的二进制科技而已，应用了高位为 1 表示负数的科技，转换代码是当 unitExponent 大于 7 时，采用 `unitExponent = unchecked((short)(0xFFF0 | unitExponent))` 转换方法。具体的转换 unitExponent 的代码如下

```csharp
if (StylusPointPropertyUnitHelper.FromPointerUnit(widthProperty.unit) ==
    StylusPointPropertyUnit.Centimeters)
{
    var unitExponent = (int) widthProperty.unitExponent;

    // 根据 HID 规范，单位指数的值范围是 0x00-0x0F，带上 mask 可以强行约束范围
    const byte HidExponentMask = 0x0F;
    // HID hut1_6.pdf 23.18.4 Generic Unit Exponent
    // 以下代码也能从 WPF 的 System.Windows.Input.StylusPointer.PointerStylusPointPropertyInfoHelper 找到
    unitExponent = (byte) (unitExponent & HidExponentMask) switch
    {
        5 => 5,
        6 => 6,
        7 => 7,
        8 => -8,
        9 => -7,
        0x0A => -6,
        0x0B => -5,
        0x0C => -4,
        0x0D => -3,
        0x0E => -2,
        0x0F => -1,
        _ => unitExponent
    };
    // 也可以这么写，正好也是相同的值。只是这么写在玩二进制的转换，不如打一个表好
    // - unchecked((short) (0xFFF0 | 0xA)) == -6
    // - unchecked((short) (0xFFF0 | 0x9)) == -7
    //if (unitExponent > 7)
    //{
    //    unitExponent = unchecked((short)(0xFFF0 | unitExponent));
    //}

    // 宽度高度都使用相同的单位值好了，预计也没有哪个厂商的触摸框有这么有趣，宽度和高度分别采用不同的单位
    var exponent = Math.Pow(10, unitExponent);

    ... // 忽略其他代码
}
```

将转换的 unitExponent 作为 10 的 N 次方，即可计算出 exponent 的值。我这里直接将宽度高度采用相同的 exponent 值，这是不准确的，但预计也没有哪个厂商的触摸框有这么有趣，宽度和高度分别采用不同的单位

既然比例已经拿到了，那么在有上文计算好的 `widthScale` 和 `heightScale` 宽度高度比例值下，直接求和物理值的比例再乘以 `exponent` 比例即可计算出物理尺寸，代码如下

```csharp
var widthPhysical = widthScale * (widthProperty.physicalMax - widthProperty.physicalMin)
*
                    exponent;
var heightPhysical = heightScale * (heightProperty.physicalMax - heightProperty.physicalMin) *
                     exponent;

rawPointerPoint = rawPointerPoint with
{
    // 物理尺寸的计算能够保持和 WPF 的 StylusPoint 拿到的相同
    PhysicalWidth = widthPhysical,
    PhysicalHeight = heightPhysical,
};
```

在 WPF 里面物理尺寸可以通过 StylusPointPropertyInfo 的 Resolution 计算而来，如以下放在 StylusMove 事件监听里的代码，获取首个 StylusPoint 的 WPF 单位宽度和物理宽度。直接通过 `stylusPointCollection[0].GetPropertyValue(StylusPointProperties.Width)` 拿到的 `width` 就是 WPF 单位的宽度，通过 `width/stylusPointPropertyInfo.Resolution` 就去可以获取 `stylusPointPropertyInfo.Unit` 单位的物理尺寸

```csharp
    private void MainWindow_StylusMove(object sender, StylusEventArgs e)
    {
        var stylusPointCollection = e.GetStylusPoints(null);
        if (stylusPointCollection.Description.HasProperty(StylusPointProperties.Width))
        {
            StylusPointPropertyInfo? stylusPointPropertyInfo = stylusPointCollection.Description.GetPropertyInfo(StylusPointProperties.Width);
            var width = stylusPointCollection[0].GetPropertyValue(StylusPointProperties.Width);

            TouchInfoTextBlock.Text +=
                $" Width=[Value:{width},Max:{stylusPointPropertyInfo.Maximum},Min:{stylusPointPropertyInfo.Minimum},Resolution:{stylusPointPropertyInfo.Resolution:0.###},Physical:{width/stylusPointPropertyInfo.Resolution:0.###}{stylusPointPropertyInfo.Unit}]";
        }
    }
```

尝试运行代码，可以看到从 WM_Pointer 指针消息解析到的宽度高度尺寸，与从 WPF 的 StylusMove 事件拿到的相同，这能够证明以上的解析代码是正确的

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c779f7cdf7970ede4fa31f73ff9f92228c7c1498/WPFDemo/NawrernalgarGibehayle) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/c779f7cdf7970ede4fa31f73ff9f92228c7c1498/WPFDemo/NawrernalgarGibehayle) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c779f7cdf7970ede4fa31f73ff9f92228c7c1498
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c779f7cdf7970ede4fa31f73ff9f92228c7c1498
```

获取代码之后，进入 WPFDemo/NawrernalgarGibehayle 文件夹，即可获取到源代码
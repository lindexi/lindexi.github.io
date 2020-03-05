# WPF 从触摸消息转触摸事件

在 WPF 程序可能因为一些坑让程序触摸失效，如果此时还可以收到系统的触摸消息，那么可以通过从触摸消息转触摸事件解决程序触摸失效但不适合所有触摸失效程序

<!--more-->
<!-- CreateTime:2019/11/29 8:47:55 -->

<!-- csdn -->

在 WPF 的触摸代码写的不是很清真，特别是[触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )可能出现一些坑，如[WPF 在触摸线程等待主线程窗口关闭会让主线程和触摸线程相互等待](https://blog.lindexi.com/post/wpf-%E5%9C%A8%E8%A7%A6%E6%91%B8%E7%BA%BF%E7%A8%8B%E7%AD%89%E5%BE%85%E4%B8%BB%E7%BA%BF%E7%A8%8B%E7%AA%97%E5%8F%A3%E5%85%B3%E9%97%AD%E4%BC%9A%E8%AE%A9%E4%B8%BB%E7%BA%BF%E7%A8%8B%E5%92%8C%E8%A7%A6%E6%91%B8%E7%BA%BF%E7%A8%8B%E7%9B%B8%E4%BA%92%E7%AD%89%E5%BE%85 ) 和 [WPF 插拔触摸设备触摸失效](https://blog.lindexi.com/post/wpf-%E6%8F%92%E6%8B%94%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88 ) 等，有时候在开机的过程，如果启动快了，触摸设备还没准备好，刚好在 WPF 初始化的过程 USB 触摸设备才准备好，此时 WPF 也会触摸失效

在希沃的设备通过判断用户的开机启动时间，如果启动时间过短，那么就需要多判断是不是 USB 设备还没准备好，如果 USB 还没准备好，那么通过一些黑科技告诉用户重新启动。因为在希沃的设备上主要是触摸屏幕，用户不会有鼠标，如果出现了初始化的过程刚好就是 USB 准备好，那么这个程序将收不到任何触摸事件

在程序启动的时候，可以通过[获得触摸精度和触摸点](https://blog.lindexi.com/post/WPF-%E8%8E%B7%E5%BE%97%E8%A7%A6%E6%91%B8%E7%B2%BE%E5%BA%A6%E5%92%8C%E8%A7%A6%E6%91%B8%E7%82%B9.html )判断当前是否存在触摸设备，如果不存在触摸设备同时判断是在希沃的设备上运行，那么就是触摸失效了。但是还可以收到系统的触摸消息，可以通过本文的黑科技收到触摸

在 WPF 的框架，触摸是从 PENIMC 里面获取的，如果通过自己创建一个模拟的触摸设备，请看 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html) 也可以做到模拟一个触摸。在默认的 WPF 程序是收不到系统的触摸消息，需要[禁用实时触摸](https://blog.lindexi.com/post/wpf-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8 )才可以收到触摸消息，在 Win7 和之后都可以从系统收到 `WM_TOUCH` 消息，通过这个消息可以解析当前的触摸点和触摸面积，通过这两个值可以用来模拟触摸走原有的 WPF 触摸

在使用 `WM_TOUCH` 消息需要用到一些本地的方法，先定义一个 NativeMethods 类，用来放本地方法

```csharp
    internal static class NativeMethods
    {
        public const int WM_TOUCH = 0x0240;
        public const uint TWF_WANTPALM = 0x00000002;

        [DllImport("user32")]
        [return: MarshalAs(UnmanagedType.Bool)]
        public static extern bool RegisterTouchWindow(IntPtr hWnd, uint ulFlags);

        [DllImport("user32")]
        [return: MarshalAs(UnmanagedType.Bool)]
        public static extern bool GetTouchInputInfo(IntPtr hTouchInput, int cInputs,
            [In, Out] TOUCHINPUT[] pInputs, int cbSize);

        [DllImport("user32")]
        [return: MarshalAs(UnmanagedType.Bool)]
        public static extern void CloseTouchInputHandle(IntPtr lParam);
    }
```

因为这个类的定义方法比较多，所以就不在本文告诉大家，请看源代码

在开启触摸消息之前需要在 Window 的 SourceInitialized 事件触发之后才能调用

创建 MessageTouchDevice 继承 TouchDevice 从 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html) 可以知道这个类可以用来模拟触摸，在这个类添加一个静态的方法 UseMessageTouch 用它传入窗口

```csharp
        public MainWindow()
        {
            InitializeComponent();

            SourceInitialized += MainWindow_SourceInitialized;
        }

        private void MainWindow_SourceInitialized(object sender, EventArgs e)
        {
             MessageTouchDevice.UseMessageTouch(this);
        }
```

在 UseMessageTouch 方法需要先通过[禁用实时触摸](https://blog.lindexi.com/post/wpf-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8 )然后使用钩子拿到消息

```csharp
       /// <summary>
        /// 使用消息触摸
        /// 注意 开启了消息触摸之后，原有的 WPF 触摸将会无法再次使用
        /// </summary>
        public static void UseMessageTouch(Window window)
        {
            // 先禁用 WPF 触摸
            TabletHelper.DisableWPFTabletSupport(hWnd);

            NativeMethods.RegisterTouchWindow(hWnd, NativeMethods.TWF_WANTPALM);
            HwndSource source = HwndSource.FromHwnd(hWnd);

            source.AddHook((IntPtr hwnd, int msg, IntPtr param, IntPtr lParam, ref bool handled) =>
            {
                WndProc(window, msg, param, lParam, ref handled);
                return IntPtr.Zero;
            });
        }
```

定义 WndProc 静态方法用来收到消息，通过消息 msg 可以判断当前是否触摸消息，然后通过 wParam 计算出当前的触摸收集到的次数

因为 Windows 消息触发比较慢，也就是没有 PENIMC 拿到触摸点那么快，在一次触发的时候可以拿到多个触摸输入

```csharp
       private static void WndProc(Window window, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            if (msg == NativeMethods.WM_TOUCH)
            {
            }
        }
```

通过下面代码，可以找到当前的消息有多少次输入

```csharp
var inputCount = wParam.ToInt32() & 0xffff;
```

然后创建一个数组，从 GetTouchInputInfo 获取所有的输入

```csharp
    var inputs = new NativeMethods.TOUCHINPUT[inputCount];

    NativeMethods.GetTouchInputInfo(lParam, inputCount, inputs, NativeMethods.TouchInputSize);
```

如果可以拿到输入，那么 GetTouchInputInfo 将会返回 true 通过这个判断

然后遍历 inputs 输入进行转换事件，从 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html) 找到通过封装的 Down 等方法可以转换为事件，请看[代码](https://github.com/lindexi/lindexi_gd/blob/f0f872153ed07b2141b47580a74a18a38cc56cfd/DernijacallqaNaycerejerlal/DernijacallqaNaycerejerlal/MainWindow.xaml.cs#L72)

在 GetTouchInputInfo 方法拿到的输入的类包含了当前触摸的屏幕坐标和触摸的面积，拿到的数据其实是原有是的百分之一也就是需要除以100才是像素

```csharp
        [StructLayout(LayoutKind.Sequential)]
        internal struct TOUCHINPUT
        {
            /// <summary>
            /// 触控输入的 X 坐标（水平点）。此成员用物理屏幕坐标的像素的百分之一表示
            /// </summary>
            public int X;

            /// <summary>
            /// 触控输入的 y 坐标（垂直点）。此成员用物理屏幕坐标的像素的百分之一表示
            /// </summary>
            public int Y;

            /// <summary>
            /// 源输入设备的设备句柄。触控输入提供程序在运行时为每个设备指定一个唯一的提供程序
            /// </summary>
            public IntPtr Source;

            /// <summary>
            /// 一个用于区别某个特定触控输入的触控点标识符。此值在触控点序列中从触控点下降到重新上升的整个过程中保持一致。稍后可对后续触控点重用一个 ID
            /// </summary>
            public int DwID;

            /// <summary>
            /// 用于指定触控点按住、释放和移动的各个方面
            /// </summary>
            public TOUCHEVENTF DwFlags;

            /// <summary>
            /// 指定结构中包含有效值的可选字段。可选字段中的有效信息的可用性是特定于设备的
            /// </summary>
            public TOUCHINPUTMASK DwMask;

            /// <summary>
            /// 事件的时间戳（以毫秒为单位）。使用方应用程序应通知系统不对此字段进行验证
            /// </summary>
            public int DwTime;

            public IntPtr DwExtraInfo;

            /// <summary>
            /// 触控区域的宽度用物理屏幕坐标的像素的百分之一表示。只有在 <see cref="DwMask"/> 成员设置了 TOUCHEVENTFMASK_CONTACTAREA 标记的情况下，此值才会有效
            /// </summary>
            public int CxContact;

            /// <summary>
            /// 触控区域的高度用物理屏幕坐标的像素的百分之一表示。只有在 <see cref="DwMask"/> 成员设置了 TOUCHEVENTFMASK_CONTACTAREA 标记的情况下，此值才会有效
            /// </summary>
            public int CyContact;
        }
```

通过下面代码可以将 TOUCHINPUT 转换为屏幕坐标和触摸面积，注意这里没有处理任何 DPI 相关，也就是我认为当前的屏幕是 96 的 DPI 的时候下面的转换的就是相对屏幕的坐标

```csharp
    var position = new Point(input.X / 100.0, input.Y / 100.0);
    var size = new Size(input.CxContact / 100.0, input.CyContact / 100.0);
```

在一次触摸的过程，需要使用相同的 TouchDevice 于是在按下和移动等需要有一个相同的实例，通过创建一个静态的字典按照触摸的 id 存放

```csharp
        private static readonly Dictionary<int, MessageTouchDevice>
            _devices = new Dictionary<int, MessageTouchDevice>();
```

在判断没有存在设备的时候创建

```csharp
if (!_devices.TryGetValue(input.DwID, out var device))
{
    device = new MessageTouchDevice(input.DwID, window);
    _devices.Add(input.DwID, device);
}
```

在判断是按下的时候触发按下

```csharp
if (!device.IsActive && input.DwFlags.HasFlag(NativeMethods.TOUCHEVENTF.TOUCHEVENTF_DOWN))
{
    device.Position = position;
    device.Size = size;
    device.Down();
}
```

其他事件也差不多，另外在 GetTouchPoint 方法需要做一点修改，添加属性 Position 和 Size 在获取的时候返回

```csharp
        /// <summary>
        /// 触摸点
        /// </summary>
        private Point Position { set; get; }

        /// <summary>
        /// 触摸大小
        /// </summary>
        private Size Size { set; get; }

        public override TouchPoint GetTouchPoint(IInputElement relativeTo)
        {
            return new TouchPoint(this, Position, new Rect(Position, Size), TouchAction);
        }
```

上面代码没有按照约定，返回输入元素相对的坐标，而是返回屏幕的坐标，所以请小伙伴自己修改代码才能在项目使用，同时因为使用的是屏幕的坐标，所以在主窗口触摸的时候，如果判断当前的触摸点在屏幕之外，那么就不会触发主窗口的触摸。因为主窗口期望的是返回的输入的点是相对的主窗口的坐标而不是相对于屏幕的坐标

所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/463a4610393090eac60a06788faa3852589a6aa2/DernijacallqaNaycerejerlal/DernijacallqaNaycerejerlal) 欢迎小伙伴帮忙修改

除了通过 Touch 消息之外，在 Win7 以上的系统，如 Window 10 系统支持 Pointer 消息，可以通过 [把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html ) 将触摸消息转 Pointer 消息进行模拟

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

# WPF 使用 RawInput 接收裸数据

在 Windows 提供很底层的方法接收硬件设备的裸数据，通过接收裸数据可以做到性能更高的全局键盘，还能支持多个鼠标。但是用这个方法需要自己解析裸数据，同时会因为接受到很多消息降低性能

<!--more-->
<!-- CreateTime:2019/11/23 16:41:52 -->

<!-- csdn -->

在微软官方很少有文档说如何使用[Raw Input](https://docs.microsoft.com/en-us/windows/win32/inputdev/about-raw-input )不过我在 github 上找到小伙伴的 [rawinput-sharp: C# wrapper library for Raw Input](https://github.com/mfakane/rawinput-sharp ) 项目，简单通过 NuGet 安装就能使用

使用 NuGet 安装 [RawInput.Sharp 0.0.2](https://www.nuget.org/packages/RawInput.Sharp ) 如果是新项目可以使用下面代码

```csharp
        <PackageReference Include="RawInput.Sharp" Version="0.0.2" />
```

在 MainWindows 注册事件，请看代码

```csharp
        public MainWindow()
        {
            InitializeComponent();

            SourceInitialized += MainWindow_SourceInitialized;
        }

        private void MainWindow_SourceInitialized(object sender, EventArgs e)
        {
            var windowInteropHelper = new WindowInteropHelper(this);
            var hwnd = windowInteropHelper.Handle;

            // Get the devices that can be handled with Raw Input.
            var devices = RawInputDevice.GetDevices();

            // register the keyboard device and you can register device which you need like mouse
            RawInputDevice.RegisterDevice(HidUsageAndPage.Keyboard,
                RawInputDeviceFlags.ExInputSink | RawInputDeviceFlags.NoLegacy, hwnd);

            HwndSource source = HwndSource.FromHwnd(hwnd);
            source.AddHook(Hook);
        }
```

通过 RawInputDevice.GetDevices 可以知道当前可以注册的设备有哪些，使用 RawInputDevice.RegisterDevice 可以注册事件，这里注册的是键盘事件，小伙伴自己修改 HidUsageAndPage 的值可以注册不同的事件

注册事件就可以在 Hook 函数接收到 WM_INPUT 消息，通过这个消息解析就可以拿到裸数据，对裸数据处理就可以收到输入，如果需要接入 WPF 可以使用[WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html )将收到的消息模拟触摸

```csharp
        private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            const int WM_INPUT = 0x00FF;

            // You can read inputs by processing the WM_INPUT message.
            if (msg == WM_INPUT)
            {
                // Create an RawInputData from the handle stored in lParam.
                var data = RawInputData.FromHandle(lparam);

                // You can identify the source device using Header.DeviceHandle or just Device.
                var sourceDeviceHandle = data.Header.DeviceHandle;
                var sourceDevice = data.Device;

                // The data will be an instance of either RawInputMouseData, RawInputKeyboardData, or RawInputHidData.
                // They contain the raw input data in their properties.
                switch (data)
                {
                    case RawInputMouseData mouse:
                        Debug.WriteLine(mouse.Mouse);
                        break;
                    case RawInputKeyboardData keyboard:
                        Debug.WriteLine(keyboard.Keyboard);
                        break;
                    case RawInputHidData hid:
                        Debug.WriteLine(hid.Hid);
                        break;
                }
            }

            return IntPtr.Zero;
        }
```

用 RawInput 就是通过 [RegisterRawInputDevices](https://docs.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-registerrawinputdevices?redirectedfrom=MSDN ) 告诉系统当前进程需要支持裸数据，系统将会根据传入的参数将裸数据转发给应用。应用在消息解析数据拿到裸数据，然后按照业务解析裸数据。这个方法可以解决一些特殊设备支持，因为 HID 设备是独占设备，只能让系统独占，如果想要应用也接收硬件发过来的消息，就需要额外通道给应用。另外应用如果需要解决其他应用钩了消息，可以注册裸数据解决其他应用勾了键盘消息

本文的例子代码在 [github](https://github.com/mfakane/rawinput-sharp/pull/5) 欢迎小伙伴访问

现在这个项目只支持 dotnet standard 2.0 我将这个项目升级兼容 .NET 4.5 我提交了 MR 请看 [Pull Request #3 rawinput-sharp](https://github.com/mfakane/rawinput-sharp/pull/3 ) 如何合并了就能兼容

[Using Raw Input](https://docs.microsoft.com/en-us/windows/win32/inputdev/using-raw-input )

[About Raw Input](https://docs.microsoft.com/en-us/windows/win32/inputdev/about-raw-input )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

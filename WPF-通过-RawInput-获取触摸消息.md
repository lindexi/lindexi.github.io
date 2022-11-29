
# WPF 通过 RawInput 获取触摸消息

触摸在 Windows 下属于比较特殊的输入，不同于键盘和鼠标，键盘和鼠标可以通过全局 Hook 的方式获取到鼠标和键盘的输入消息。而触摸则没有直接的 Hook 的方法。如果期望自己的应用，可以在没有作为前台获取焦点的应用时，可以抓取到全局的触摸消息，抓取到其他应用程序的触摸输入，那么可以尝试使用 RawInput 的方式。通过 RawInput 的方式，可以让一个没有任何激活的、触摸直接命中的窗口的应用程序接收到全局的所有触摸消息

<!--more-->


<!-- CreateTime:2022/11/25 20:00:16 -->


<!-- csdn -->
<!-- 博客 -->
<!-- 发布 -->

在上一篇博客，介绍了 [WPF 使用 RawInput 接收裸数据](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-RawInput-%E6%8E%A5%E6%94%B6%E8%A3%B8%E6%95%B0%E6%8D%AE.html ) 的方法，但是里面只是和大家演示了如何抓取鼠标和键盘消息

其实通过 RawInput 是可以在注册设备时，声明需要获取 TouchScreen 触摸屏输入，和 Pen 笔输入的。如以下代码

```csharp
        RawInputDevice.RegisterDevice(HidUsageAndPage.TouchScreen,
            RawInputDeviceFlags.ExInputSink | RawInputDeviceFlags.DevNotify, hwnd);
        RawInputDevice.RegisterDevice(HidUsageAndPage.Pen,
            RawInputDeviceFlags.ExInputSink | RawInputDeviceFlags.DevNotify, hwnd);
```

以上的代码有所省略，没关系，大家可以在本文最后获取到所有的源代码

注册完成之后，即可在消息循环里面，收到 Windows 调度的消息。基于 [rawinput-sharp: C# wrapper library for Raw Input](https://github.com/mfakane/rawinput-sharp ) 项目的 [RawInput.Sharp](https://www.nuget.org/packages/RawInput.Sharp ) 库的自动转换，即可通过判断 RawInputHidData 的方式，过滤出触摸的输入，代码如下

```csharp
    private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {
        const int WM_INPUT = 0x00FF;

        // You can read inputs by processing the WM_INPUT message.
        if (msg == WM_INPUT)
        {
            // Create an RawInputData from the handle stored in lParam.
            var data = RawInputData.FromHandle(lparam);

            // The data will be an instance of either RawInputMouseData, RawInputKeyboardData, or RawInputHidData.
            // They contain the raw input data in their properties.
            switch (data)
            {
                case RawInputHidData hid:
                    Debug.WriteLine($"Hid {hid.Hid}");
                    TextBlock.Text = @$"DevicePath: {hid.Device.DevicePath}
VID:{hid.Device.VendorId:X2} PID:{hid.Device.ProductId:X2}
RawData:{hid.Hid}

";
                    if (hid is RawInputDigitizerData rawInputDigitizerData)
                    {
                        foreach (var rawInputDigitizerContact in rawInputDigitizerData.Contacts)
                        {
                            TextBlock.Text += rawInputDigitizerContact.ToString() + "\r\n";
                        }
                    }
                    break;
            }
        }

        return IntPtr.Zero;
    }
```

如上面代码，可以看到，在获取输入信息时，还可以获取到是哪个触摸框的输入，通过触摸框的 DevicePath 或者是 ProductId 等判断。同时通过 RawInput 的此方法，也可以用来支持双触摸屏同时进行触摸输入

以下是更多代码细节

在使用之前，安装 NuGet 包。如果是 SDK 风格的 csproj 项目格式，可以在项目文件添加以下代码安装

```xml
  <ItemGroup>
    <PackageReference Include="RawInput.Sharp" Version="0.0.2" />
  </ItemGroup>
```

以下是 MainWindow.xaml.cs 的例子代码

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using Linearstar.Windows.RawInput;

namespace RaicheadoherneanuNalokearwherno;
/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        SourceInitialized += MainWindow_SourceInitialized;
    }

    private void MainWindow_SourceInitialized(object? sender, EventArgs e)
    {
        var windowInteropHelper = new WindowInteropHelper(this);
        var hwnd = windowInteropHelper.Handle;

        // Get the devices that can be handled with Raw Input.
        var devices = RawInputDevice.GetDevices();

        // register the keyboard device and you can register device which you need like mouse
        RawInputDevice.RegisterDevice(HidUsageAndPage.Keyboard,
            RawInputDeviceFlags.ExInputSink | RawInputDeviceFlags.NoLegacy, hwnd);

        RawInputDevice.RegisterDevice(HidUsageAndPage.TouchScreen,
            RawInputDeviceFlags.ExInputSink | RawInputDeviceFlags.DevNotify, hwnd);
        RawInputDevice.RegisterDevice(HidUsageAndPage.Pen,
            RawInputDeviceFlags.ExInputSink | RawInputDeviceFlags.DevNotify, hwnd);

        HwndSource source = HwndSource.FromHwnd(hwnd);
        source.AddHook(Hook);
    }

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
                    Debug.WriteLine($"Mouse {mouse.Mouse}");
                    break;
                case RawInputKeyboardData keyboard:
                    Debug.WriteLine(keyboard.Keyboard);
                    break;
                case RawInputHidData hid:
                    Debug.WriteLine($"Hid {hid.Hid}");
                    TextBlock.Text = @$"DevicePath: {hid.Device.DevicePath}
VID:{hid.Device.VendorId:X2} PID:{hid.Device.ProductId:X2}
RawData:{hid.Hid}

";
                    if (hid is RawInputDigitizerData rawInputDigitizerData)
                    {
                        foreach (var rawInputDigitizerContact in rawInputDigitizerData.Contacts)
                        {
                            TextBlock.Text += rawInputDigitizerContact.ToString() + "\r\n";
                        }
                    }
                    break;
            }
        }

        return IntPtr.Zero;
    }
}
```


本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/bc08596b5f3b261ff621fcdb1426c953e3514b69/RaicheadoherneanuNalokearwherno ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bc08596b5f3b261ff621fcdb1426c953e3514b69/RaicheadoherneanuNalokearwherno ) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bc08596b5f3b261ff621fcdb1426c953e3514b69 
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bc08596b5f3b261ff621fcdb1426c953e3514b69 
```

获取代码之后，进入 RaicheadoherneanuNalokearwherno  文件夹

更多 WPF 相关博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# WPF 编写一个测试 WM_TOUCH 触摸消息延迟的应用

我听说在 Win10 到 Win11 的系统版本左右，微软加上了一大波触摸性能优化，准确来说是 HID 性能优化。我想测试一下在这些系统下，采用从 Windows 消息接收到 WM_TOUCH 触摸消息的延迟将会是多少。本文将告诉大家我编写的测试应用

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

为了能够让 WPF 窗口能接收到 WM_TOUCH 触摸消息，首先需要将 WPF 默认走的实时触摸机制禁用，否则两个触摸接收方法将会打架，在 Windows 层将不会调度 WM_TOUCH 触摸消息给到 WPF 窗口。根据 [WPF 禁用实时触摸](https://blog.lindexi.com/post/WPF-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8.html ) 提供的方法禁用实时触摸，如果没有禁用 WPF 的 RealTimeStylus 实时触摸，就无法拿到 WM_TOUCH 消息，这是因为两套触摸机制将会打架。在 Windows 系统层发现开启了实时触摸之后，将不会调度 WM_TOUCH 消息给到应用窗口

在 App 构造函数加上以下代码用来禁用 RealTimeStylus 实时触摸

```csharp
public partial class App : Application
{
    public App()
    {
        AppContext.SetSwitch("Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport", true);
    }
}
```

为了更加方便调用 Win32 函数，按照 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html ) 博客的方法，安装 Microsoft.Windows.CsWin32 库用来减少编写 PInvoke 的定义方法

这里采用 .NET 7 的 WPF 项目，可以编辑 csproj 用来安装 Microsoft.Windows.CsWin32 库，十分方便，修改 csproj 项目文件为以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net7.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Windows.CsWin32" PrivateAssets="all" Version="0.2.63-beta" />
  </ItemGroup>
</Project>
```

根据 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html ) 博客提到的方法，需要在项目新建一个名为 NativeMethods.txt 的文件，在此文件里面写入需要使用的 Win32 函数。此时 CsWin32 库将使用 SourceGenerator 技术生成这些写入的 Win32 函数的定义

下面是在 NativeMethods.txt 写入的代码

```
RegisterTouchWindow
GetMessageTime
GetTouchInputInfo
```

进入到 MainWindow 函数里面监听 SourceInitialized 事件。在 WPF 框架里面，约定了在 SourceInitialized 事件里就是创建完成了 Win32 窗口之后触发的，在此事件里面使用 Win32 窗口相关方法是安全的

```csharp
    public MainWindow()
    {
        InitializeComponent();

        SourceInitialized += MainWindow_SourceInitialized;
    }

    private void MainWindow_SourceInitialized(object? sender, EventArgs e)
    {
    }
```

在 `MainWindow_SourceInitialized` 方法里面调用 RegisterTouchWindow 用来注册 WM_Touch 消息，代码如下

```csharp
        var windowInteropHelper = new WindowInteropHelper(this);
        var hwnd = windowInteropHelper.Handle;

        // 如果启用了 TWF_WANTPALM ，则不会缓冲触摸输入中的数据包，并且不会在将数据包发送到应用程序之前执行手掌检测。 如果要在处理 WM_TOUCH 消息时实现最小延迟，则启用 TWF_WANTPALM 最有用
        PInvoke.RegisterTouchWindow(new HWND(hwnd), REGISTER_TOUCH_WINDOW_FLAGS.TWF_WANTPALM);
```

这里传入了 `TWF_WANTPALM` 参数，传入这个参数可以减少触摸消息延迟

接着根据 [WPF 添加窗口消息钩子方法](https://blog.lindexi.com/post/WPF-%E6%B7%BB%E5%8A%A0%E7%AA%97%E5%8F%A3%E6%B6%88%E6%81%AF%E9%92%A9%E5%AD%90%E6%96%B9%E6%B3%95.html ) 博客接收 Windows 消息，代码如下

```csharp
    private void MainWindow_SourceInitialized(object? sender, EventArgs e)
    {
        var windowInteropHelper = new WindowInteropHelper(this);
        var hwnd = windowInteropHelper.Handle;

        // 如果启用了 TWF_WANTPALM ，则不会缓冲触摸输入中的数据包，并且不会在将数据包发送到应用程序之前执行手掌检测。 如果要在处理 WM_TOUCH 消息时实现最小延迟，则启用 TWF_WANTPALM 最有用
        PInvoke.RegisterTouchWindow(new HWND(hwnd), REGISTER_TOUCH_WINDOW_FLAGS.TWF_WANTPALM);

        HwndSource source = HwndSource.FromHwnd(hwnd)!; // 这里在 SourceInitialized 一定是存在的
        source.AddHook(Hook);
    }

    private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {
        // 忽略代码
    }
```

在 Hook 函数里面，判断收到的消息是否 WM_Touch 消息，如果是那就记录当前的消息时间，用来判断两条 WM_Touch 消息之间的延迟

```csharp
    private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {
        if (msg == WM_TOUCH)
        {
            // 触摸进来的
            var currentMessageTime = PInvoke.GetMessageTime();
            if (_lastTouchMessageTime != 0)
            {
                var delay = currentMessageTime - _lastTouchMessageTime;
                // 这就是消息的延迟
            }

            _lastTouchMessageTime = currentMessageTime;
        }

        return IntPtr.Zero;
    }

    private int _lastTouchMessageTime;

    private const int WM_TOUCH = 0x0240;
```

在自己的电脑上运行代码，即可用来测试 WM_Touch 触摸的延迟

我使用以上代码在我的 Demo 上测试和在我的一个复杂项目上测试，结果就是在 Demo 上的触摸延迟是 WM_Touch 和 RealTimeStylus 实时触摸几乎一样。但是在复杂的项目上，由于 Windows 消息太多或者是主线程忙碌，触摸延迟是 WM_Touch 比 RealTimeStylus 实时触摸大许多

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/f006732c9f97f370f5c063de024125b201313bd3/WalqujemjelNekokelhuwererere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f006732c9f97f370f5c063de024125b201313bd3/WalqujemjelNekokelhuwererere) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f006732c9f97f370f5c063de024125b201313bd3
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f006732c9f97f370f5c063de024125b201313bd3
```

获取代码之后，进入 WalqujemjelNekokelhuwererere 文件夹

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )

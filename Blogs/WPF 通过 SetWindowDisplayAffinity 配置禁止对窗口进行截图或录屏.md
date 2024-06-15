有些应用程序比较机密或隐私，不期望被其他截图软件截图到应用的窗口，或者被录屏软件录制到。简单的方法是通过 SetWindowDisplayAffinity 方法进行配置窗口阻止截图软件对其截图

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

开始之前必须说明的是对抗截图录屏是一个矛和盾的事情，截图和录屏技术方向在千方百计尝试对所有窗口进行截图和录屏。而某些机密或隐私等软件又在对抗截图和录屏。本文使用的 SetWindowDisplayAffinity 只是一个非常基础的禁止窗口被截图的方法，能防住的截图工具和录屏软件有限，只能做简单的保护窗口不被基础截图工具所获取界面

按照使用 Win32 方法的惯例，先定义出来 SetWindowDisplayAffinity 方法，代码如下

```csharp
    private const uint WDA_NONE = 0x00000000;
    private const uint WDA_MONITOR = 0x00000001;

    [DllImport("user32.dll")]
    public static extern uint SetWindowDisplayAffinity(IntPtr hWnd, uint dwAffinity);
```

在 .NET 7 之后，还可以使用 LibraryImportAttribute 这个源代码生成器辅助的定义 Win32 方法，对比 DllImport 的优势在于能够通过源代码生成器优化调用的性能。更新之后的定义代码如下，核心是将 `extern` 换成 `partial` 关键词和更换标记的特性

```csharp
    [LibraryImport("user32.dll")]
    public static partial uint SetWindowDisplayAffinity(IntPtr hWnd, uint dwAffinity);
```

对于本文使用的如此简单的 SetWindowDisplayAffinity 方法，使用 LibraryImportAttribute 是没有带来什么好处的

且使用此特性需要给当前的项目开启不安全代码的允许开关。更多请参考 [P/Invoke source generation - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke-source-generation )

为了方便本文描述，我新建了一个例子项目，可以在本文末尾找到本文所有代码的下载方法

在 MainWindow.xaml 放一个按钮，用于控制设置窗口允许和禁止截图的状态

```xml
        <ToggleButton x:Name="TakeSnapshotToggleButton" HorizontalAlignment="Center" VerticalAlignment="Center" Padding="10,10,10,10" Content="禁止截图" Checked="TakeSnapshotToggleButton_OnChecked"/>
```

后台代码编写实现逻辑

```csharp
    private void TakeSnapshotToggleButton_OnChecked(object sender, RoutedEventArgs e)
    {
        if (TakeSnapshotToggleButton.IsChecked is true)
        {
            // 禁止截图模式
            SetWindowDisplayAffinity(new WindowInteropHelper(this).Handle, WDA_MONITOR);

            // 修改内容为再点击就是允许截图
            TakeSnapshotToggleButton.Content = "允许截图";
        }
        else
        {
            // 允许截图模式
            SetWindowDisplayAffinity(new WindowInteropHelper(this).Handle, WDA_NONE);

            // 修改内容为再点击就是禁止截图
            TakeSnapshotToggleButton.Content = "禁止截图";
        }
    }
```

如此即可实现此按钮功能，尝试运行代码，点击按钮，进入禁止截图状态。然后使用截图软件，如 QQ 截图等工具尝试进行截图，可以看到窗口是黑的不能被截图

接着再点击按钮，进入允许截图状态，此时可以看到截图软件可以对窗口进行截图可以看到窗口的内容

通过本文的方法只能防御有限的截图软件。有些从驱动级进行获取界面图像的，或者 Hook 掉 DWM 的，甚至更彻底的从 HDMI 级硬件捕获的，这些都统统无法防御

在 Windows 10 的 2004 版本，对 SetWindowDisplayAffinity 方法进行了扩展，添加了只允许在显示器显示而不在任何截图或录屏工具显示的参数。在原先的 SetWindowDisplayAffinity 使用 `WDA_MONITOR` 禁止截图时，使用截图工具将看到一个黑色的窗口，看不到任何内容。但是对于一些录屏软件来说，会影响其体验。有时候期望做一个录屏辅助工具，却要么发现录屏辅助工具被录屏工具录制进去，要么就是黑色一片影响交互。通过新的 WDA_EXCLUDEFROMCAPTURE 参数，可以有效进行优化

使用 WDA_EXCLUDEFROMCAPTURE 参数，可以配置应用窗口只允许在显示器显示而不在任何截图或录屏工具显示，这就意味着窗口对于截图软件录屏软件来说是隐藏的，从截图软件里面不再可以看到应用窗口，截图软件不会看到黑色的窗口而是完全不知道有这样的窗口的存在

使用方法也非常简单，如以下代码

```csharp
SetWindowDisplayAffinity(new WindowInteropHelper(this).Handle, WDA_EXCLUDEFROMCAPTURE);

    private const uint WDA_EXCLUDEFROMCAPTURE = 0x00000011;

    [DllImport("user32.dll")]
    public static extern uint SetWindowDisplayAffinity(IntPtr hWnd, uint dwAffinity);
```

大家可以运行代码，测试一些分别设置 WDA_MONITOR 和 WDA_EXCLUDEFROMCAPTURE 参数，对截图软件的影响

特别说明的是，只有在 Windows 10 的 2004 和更高版本，才能支持 WDA_EXCLUDEFROMCAPTURE 参数。如果在更低的版本运行，则 WDA_EXCLUDEFROMCAPTURE 参数的功能和 WDA_MONITOR 相同

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/21c1500cd1ca8bffe892644425235de7eac24f92/WPFDemo/LecacheniJequchaferenal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/21c1500cd1ca8bffe892644425235de7eac24f92/WPFDemo/LecacheniJequchaferenal) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 21c1500cd1ca8bffe892644425235de7eac24f92
```

以上使用的是 gitee 的源，如果 gitee 不能访问导致拉取失败，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 21c1500cd1ca8bffe892644425235de7eac24f92
```

获取代码之后，进入 WPFDemo/LecacheniJequchaferenal 文件夹，即可获取到源代码

## 参考文档

[P/Invoke source generation - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke-source-generation )

<https://github.com/akinbicer/screen-capture-protector>

[SetWindowDisplayAffinity function (winuser.h) - Win32 apps Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowdisplayaffinity )

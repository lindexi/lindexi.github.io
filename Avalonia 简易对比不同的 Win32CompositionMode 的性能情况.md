# Avalonia 简易对比不同的 Win32CompositionMode 的性能情况

本文对 11.3.11 版本的 Avalonia 在 4K 屏幕上进行简单的性能对比，对比不同的 Win32CompositionMode 对性能的影响情况

<!--more-->
<!-- CreateTime:2026/02/06 07:17:20 -->

<!-- 发布 -->
<!-- 博客 -->

测试代码非常简单，只是尝试修改一个控件的背景色，让界面不断更新而已

以下是 MainWindow.axaml 代码

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="JowekukurNelaholefewhi.MainWindow"
        Title="JowekukurNelaholefewhi"
        ExtendClientAreaChromeHints="NoChrome"
        Background="Transparent"
        ExtendClientAreaToDecorationsHint="False">
	<Border x:Name="BackgroundBorder">
        <Button x:Name="ChangeTransparencyLevelHintButton" Width="200" Height="100" HorizontalAlignment="Center" VerticalAlignment="Center" Click="ChangeTransparencyLevelHintButton_OnClick"
                Background="Blue">
            <Button.Content>
                <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center"
                           Foreground="White">
                    Click to Change TransparencyLevelHint
                </TextBlock>
            </Button.Content>
        </Button>
    </Border>
</Window>
```

以下是 `MainWindow.axaml.cs` 代码

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        RendererDiagnostics.DebugOverlays = RendererDebugOverlays.Fps;
        Loaded += MainWindow_Loaded;
    }

    private async void MainWindow_Loaded(object? sender, RoutedEventArgs e)
    {
        while (IsLoaded)
        {
            await Task.Delay(10);

            var color = new Color(0x02, NextByte(), NextByte(), NextByte());
            BackgroundBorder.Background = new ImmutableSolidColorBrush(color);
        }

        static byte NextByte() => (byte) Random.Shared.Next(byte.MaxValue);
    }

    private void ChangeTransparencyLevelHintButton_OnClick(object? sender, RoutedEventArgs e)
    {
        if (TransparencyLevelHint.First() == WindowTransparencyLevel.Transparent)
        {
            TransparencyLevelHint = [WindowTransparencyLevel.AcrylicBlur];
        }
        else
        {
            TransparencyLevelHint = [WindowTransparencyLevel.Transparent];
        }
    }
}
```

核心是在 `MainWindow_Loaded` 里面不断刷新界面

本文所采用的测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2599a486433e897590ba552cd9049a1bfcdf364f/AvaloniaIDemo/JowekukurNelaholefewhi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2599a486433e897590ba552cd9049a1bfcdf364f/AvaloniaIDemo/JowekukurNelaholefewhi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2599a486433e897590ba552cd9049a1bfcdf364f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2599a486433e897590ba552cd9049a1bfcdf364f
```

获取代码之后，进入 AvaloniaIDemo/JowekukurNelaholefewhi 文件夹，即可获取到源代码

不同的测试用例我没有独立代码项目，只是通过 git 的 commit 区分

最近我在摸索 Avalonia 的渲染层，这个问题源自于 7 年前，我尝试给 Avalonia 添加笔迹应用。在去年的时候，我发现 Avalonia 的笔迹性能非常糟糕，今年我设计了一个测试用例。在 Avalonia 窗口上叠加一个透明的 WPF 窗口，从 Avalonia 收到鼠标或触摸输入之后，再发送到 WPF 窗口上，让 Avalonia 和 WPF 窗口同时对一个 Border 进行 RenderTransform 平移

此测试发现了 WPF 的渲染非常跟输入，而 Avalonia 明显落后

在我的测试用例里面，特别让 Avalonia 窗口去接收输入，让 Avalonia 驱动 WPF 的界面。如此可以排除 Avalonia 的输入层带来的延迟。完全只对比 Avalonia 和 WPF 的渲染层

详细请参阅： <https://github.com/AvaloniaUI/Avalonia/discussions/20562>

实验情况如下图所示，蓝色为 Avalonia 的控件，红色是 WPF 的控件

<!-- ![](image/Avalonia 简易对比不同的 Win32CompositionMode 的性能情况/Avalonia 简易对比不同的 Win32CompositionMode 的性能情况0.gif) -->
![](http://cdn.lindexi.site/lindexi-RenderingLatency.gif)

为此，我尝试了 Avalonia 的各个 Win32CompositionMode 来摸索渲染延迟。以下是我的实验情况

本次实验的机器配置如下：

- 屏幕： 3840x2160 (4K) + 百分百 DPI
- CPU： i5-12450H
- GPU： 集显

## LowLatencyDxgiSwapChain

测试代码： <https://github.com/lindexi/lindexi_gd/tree/2599a486433e897590ba552cd9049a1bfcdf364f/AvaloniaIDemo/JowekukurNelaholefewhi>

测试结果：

  - GPU: 占用为 70-80 范围
  - DWM: 占用为 1
  - 帧率: 55-60 大部分时候靠近 60 帧率

## WinUIComposition

测试代码： <https://github.com/lindexi/lindexi_gd/tree/8c3e57108cafaf5c6ab1c0b371e62f180ba62d9b/AvaloniaIDemo/JowekukurNelaholefewhi>

  - GPU: 占用在 70 附近
  - DWM: 占用为 40 左右，可见合成过程中确实让 GPU 非常繁忙
  - 帧率: 20-30 帧

虽然 GPU 没有吃满，但是已经掉帧了，感觉这里应该有坑

## DirectComposition

测试代码： <https://github.com/lindexi/lindexi_gd/tree/6b2adfb85f1516663f128d0c8a3a7465069dbdfd/AvaloniaIDemo/JowekukurNelaholefewhi>

  - GPU: 占用在 70 附近
  - DWM: 占用为 40 左右
  - 帧率: 20-30 帧

可见 WinUIComposition 和 DirectComposition 的问题差不多

而 LowLatencyDxgiSwapChain 能够获取比较好的帧率，且对 DWM 占用比较少
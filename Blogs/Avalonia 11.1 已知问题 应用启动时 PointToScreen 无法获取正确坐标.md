本文记录 Avalonia 11.1 版本的已知问题，在 Linux 上使用 X11 时，在应用启动时，即使在 Loaded 或 Activated 事件里，都无法使用 PointToScreen 获取到正确的屏幕坐标，只会将传入的点作为返回值

<!--more-->


<!-- CreateTime:2024/08/10 07:19:10 -->

<!-- 发布 -->
<!-- 博客 -->

此问题已经报告给 Avalonia 官方，请看 <https://github.com/AvaloniaUI/Avalonia/issues/16622>

如以下代码所示

```csharp
    public MainWindow()
    {
        InitializeComponent();
        Loaded += MainWindow_Loaded;
        Activated += MainWindow_Activated;
    }

    private void MainWindow_Activated(object? sender, EventArgs e)
    {
        var pointToScreen = this.PointToScreen(new Point(0, 0));
        Console.WriteLine($"MainWindow_Activated PointToScreen={pointToScreen}");
    }

    private void MainWindow_Loaded(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {
        var pointToScreen = this.PointToScreen(new Point(0, 0));
        Console.WriteLine($"MainWindow_Loaded PointToScreen={pointToScreen}");
    }
```

将以上代码运行在 X11 上，将无法在 Loaded 或 Activated 事件里使用 PointToScreen 获取到正确的屏幕坐标

运行以上代码在 X11 上将会在控制台有以下信息

```
MainWindow_Loaded PointToScreen=0, 0
MainWindow_Activated PointToScreen=0, 0
```

如果此时在 `MainWindow_Loaded` 添加 Task.Delay 一秒即可拿到正确的屏幕坐标

```csharp
    private async void MainWindow_Loaded(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {
        var pointToScreen = this.PointToScreen(new Point(0, 0));
        Console.WriteLine($"MainWindow_Loaded PointToScreen={pointToScreen}"); // It can not get the correct coordinates here!

        await Task.Delay(1000);

        pointToScreen = this.PointToScreen(new Point(0, 0));
        Console.WriteLine(pointToScreen); // It can get the correct coordinates.
    }
```

以上问题我在 UOS 统信系统和 Kylin 麒麟系统上都进行测试，且通过分析代码可以了解到此问题与系统没有相关性。即不是 UOS 统信系统和 Kylin 麒麟系统挖的坑

此问题原因是在 Avalonia 里面依赖当前窗口坐标进行 PointToScreen 的计算，而坐标是在 X11 的 ConfigureNotify 事件里面更新的，这就意味着在窗口 Loaded 或 Activated 事件里还没有完成坐标的更新，从而导致无法正确计算屏幕坐标

由于窗口坐标更新将会触发 PositionChanged 事件，如果想要规避此问题，可以将在 Loaded 事件执行的 PointToScreen 方法尝试更改为 PositionChanged 执行，如下面代码

```csharp
    public MainWindow()
    {
        InitializeComponent();
        PositionChanged += MainWindow_PositionChanged;
    }

    private void MainWindow_PositionChanged(object? sender, PixelPointEventArgs e)
    {
        var pointToScreen = this.PointToScreen(new Point(0, 0));
        Console.WriteLine($"PositionChanged PointToScreen={pointToScreen}");
    }
```

必须说明的是 PositionChanged 和 Loaded 是完全不相同的时机，还请大家根据自己的业务进行修改

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7036c43bcea5d9057dcddfea7ff3ef7aae84dc07/AvaloniaIDemo/JejanayaYemjergayle) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7036c43bcea5d9057dcddfea7ff3ef7aae84dc07/AvaloniaIDemo/JejanayaYemjergayle) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7036c43bcea5d9057dcddfea7ff3ef7aae84dc07
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7036c43bcea5d9057dcddfea7ff3ef7aae84dc07
```

获取代码之后，进入 AvaloniaIDemo/JejanayaYemjergayle 文件夹，即可获取到源代码

更多 Avalonia 相关博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

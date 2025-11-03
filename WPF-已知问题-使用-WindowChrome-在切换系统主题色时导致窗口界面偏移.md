
# WPF 已知问题 使用 WindowChrome 在切换系统主题色时导致窗口界面偏移

本文记录 WPF 的一个已知问题，设置窗口的 WindowChrome 的 NonClientFrameEdges 属性之后，当切换系统主题或主题色时，将可以看到窗口界面内容偏移或跳动闪烁现象

<!--more-->


<!-- CreateTime:2025/11/03 08:54:05 -->

<!-- 发布 -->
<!-- 博客 -->

此问题一开始是 Office Tool Plus 的作者 [Yerong](https://github.com/YerongAI) 在 <https://github.com/dotnet/wpf/issues/11204> 报告给我的问题。我着手调查此问题时，没有去翻历史记录。于是调查路径走偏。后面在 [Michael Dietrich](https://github.com/MichaeIDietrich) 大佬的纠偏下，我才回忆起在 2020 那会，同是 WPF 开源仓库成员的 [Bastian Schmidt](https://github.com/batzen) 大佬在 <https://github.com/dotnet/wpf/issues/3193> 处理过相关的问题。[Bastian Schmidt](https://github.com/batzen) 大佬就是 [Snoop](https://github.com/snoopwpf/snoopwpf) 工具的作者

此问题不能全说是 WPF 的问题，系统层也存在一个设计如此的问题。只是在 WPF 里面也有一个锅，那就是在 <https://github.com/dotnet/wpf/blob/808031c2d49fd49ece87fb1e27d9c0774f30aa6b/src/Microsoft.DotNet.Wpf/src/PresentationFramework/System/Windows/Shell/WindowChromeWorker.cs#L445> 这里面，只是返回 `WVR.REDRAW` 重新渲染，而没有带上 `WVR_VALIDRECTS` 需要刷新脏区的

由于 WPF 需要考虑的范围太多，给 WPF 叠加这个改动估计没人敢合并，因为有啥副作用是难以快速测试出来的，影响面过广。鉴于此 [Bastian Schmidt](https://github.com/batzen) 大佬就推荐使用 <https://github.com/ControlzEx/ControlzEx> 库。我现在自己的项目或团队的项目的 WindowChrome 也都在用 <https://github.com/ControlzEx/ControlzEx> 或 <https://github.com/dotnet-campus/dotnetCampus.Windows7.Shell> 的实现

我认为对此问题的最佳解决方法是换成 <https://github.com/ControlzEx/ControlzEx> 库，或者是取出其实现代码抄过来

如果不想使用库，那也可以使用 [Michael Dietrich](https://github.com/MichaeIDietrich) 大佬 提供的方法：自己给窗口带上钩子，当收到 `WM_NCPAINT` 或 `WM_SETTINGCHANGE` 消息时，则调用 [`InvalidateRect`](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-invalidaterect) 通知系统渲染层刷新整个窗口界面。其核心代码如下

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

    private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {      
        const int WM_NCPAINT = 0x0085;
        const int WM_SETTINGCHANGE = 0x001A;
        if (msg is WM_NCPAINT or WM_SETTINGCHANGE)
        {
            Window window = this;
            var rect = new Int32Rect(0,0, (int)Math.Ceiling(window.ActualWidth), (int) Math.Ceiling(window.ActualHeight));
            unsafe
            {
                InvalidateRect(hwnd, &rect, true);
            }
        }

        return IntPtr.Zero;
    }

    [DllImport("User32")]
    private static extern unsafe bool InvalidateRect(IntPtr hwnd, Int32Rect* lpRect, bool bErase);
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9bc7db74f69823d0c0d762492dea640c8048d67c/WPFDemo/LellaibeayeelaRakearjemfal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9bc7db74f69823d0c0d762492dea640c8048d67c/WPFDemo/LellaibeayeelaRakearjemfal) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9bc7db74f69823d0c0d762492dea640c8048d67c
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9bc7db74f69823d0c0d762492dea640c8048d67c
```

获取代码之后，进入 WPFDemo/LellaibeayeelaRakearjemfal 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
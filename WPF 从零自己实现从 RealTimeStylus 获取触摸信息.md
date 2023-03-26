# WPF 从零自己实现从 RealTimeStylus 获取触摸信息

本文将告诉大家什么是 RealTimeStylus 以及如何从零开始不使用 WPF 框架提供的功能从 RealTimeStylus 获取到触摸信息

<!--more-->

<!-- 发布 -->
<!-- 博客 -->

开始之前先复习一下 Windows 的触摸演进。在上古 xp 时代，那会还没有约定好 `WM_Touch` 消息，但是那时就有了白板类应用笔迹书写的需求了。和鼠标的相对移动不相同的是，人类在触摸屏上进行书写的时候，如果书写的存在一点点延迟，大概是 50 毫秒以上，那么人类将可以明显感知到延迟。如果让每个软件开发厂商都和具体的硬件触摸厂商约定触摸需要传输的协议，自然是不靠谱的。而且也不见得每个软件厂商都能实现出比较高性能的触摸

在 XP 时代，就提出了 RealTimeStylus 实时触摸的概念。只不过那会还只是一个初始，因为那时许多触摸才起步，许多规范还没完全建立起来，或者说触摸框厂商还没学会触摸标准

在 win7 时代，引入了 `WM_Touch` 的概念，作为一个 Windows 消息，很难顶住 240Hz 以上的高刷触摸框，而且 Windows 消息本身会受到许多其他第三方应用的干扰以及业务本身的影响，导致了想要通过 `WM_Touch` 获取高性能的触摸数据，进而实现高性能书写是一个从原理上来说比较困难的事情。于是软的大佬们，又想起了 RealTimeStylus 实时触摸技术，继续完善 RealTimeStylus 机制。大概也就是从 Win7 开始，才是真正 RealTimeStylus 实时触摸强大起来的时候。因为在 XP 那会，还是有许许多多的应用都是靠私有 USB 或 HID 协议获取触摸点的，完全无法和 RealTimeStylus 对接，更不要说和 `WM_Touch` 对接

在 Win7 下，实时触摸是从一个名为 wisptis 的特殊进程，即 Windows Ink Services Platform Tablet Input Subsystem 进程进行分发的。通过微软的 Surface 触摸架构文档可以看到，这个 wisptis 特殊进程跑了一半是在内核态里面，一半在用户态里面。由于我在写这篇博客的时候，没有找出我之前看过的微软的 Surface 触摸架构文档，我怕误导大家，这里就还先跳过细节。只需要知道实时触摸是从 wisptis 的特殊进程过来的即可。这也就回答了 [为什么 WPF 软件在 win7 启动时会尝试调起 wisptis 进程](https://blog.lindexi.com/post/%E4%B8%BA%E4%BB%80%E4%B9%88-WPF-%E8%BD%AF%E4%BB%B6%E5%9C%A8-win7-%E5%90%AF%E5%8A%A8%E6%97%B6%E4%BC%9A%E5%B0%9D%E8%AF%95%E8%B0%83%E8%B5%B7-wisptis-%E8%BF%9B%E7%A8%8B.html) 这个问题

在 Win10 下，这时整个触摸行业才算是完全统一了，大家都遵循标准触摸协议。软软也根据之前踩过的坑，重新设计了整个系统的触摸架构。同时野心很大的软软为了能够支持 VR 头盔等的视线输入等，将所有输入统一，引入了 `WM_Pointer` 概念。也就是无论是鼠标还是触摸，还是视线输入，都是 Pointer 消息

只不过 Pointer 消息也是一个 Windows 消息，依然也受到 Windows 消息的限制。这里需要特别说的是，快和慢是相对的，不能说 Windows 消息是高延迟的，也不能说 Windows 消息是慢的。仅仅只是在高性能笔迹书写的情况下，在妖魔的用户环境下，以及业务的成堆的诡异代码下，才会认为走 Windows 消息是不能做到低延迟的

这里也需要额外说一个题外话，那就是触摸的延迟不仅仅只是系统层也应用层说了算了，而是一个整体的问题，从硬件本身的设计开始到应用层每个模块都能影响。说到这里就需要提一下上古的，但是现在电竞依然还在采用的 PS/2 接口。对比采用轮询设备式运行的 USB 设备，采用系统中断的 PS/2 从原理层面来说能够实现更低的延迟，实际使用的反应更快。尽管 PS/2 接口的数据传输速度不能和 USB 打，但是从低时延方面上还是能够压过 USB 设备的。现在的大尺寸触摸屏幕行业上的触摸基本都是走 USB/HID 方式，也就是从此原理上来说延迟性就存在部分了。那是不是有人就要问了，让大尺寸触摸屏幕的触摸走 PS/2 是否可以？也许可以，但是你要做好了一条龙都需要自己实现的准备。但是在小尺寸的触摸屏上，早已有一些厂商采用 PS/2 接口方式

在 Win10 改了触摸架构，但是我没有找到官方文档，同时也了解到 Win10 的 wisptis 是附加到进程的窗口，详细请看 [Win10 的 WPF 程序的 wisptis 服务是附加到进程的窗口](https://blog.lindexi.com/post/Win10-%E7%9A%84-WPF-%E7%A8%8B%E5%BA%8F%E7%9A%84-wisptis-%E6%9C%8D%E5%8A%A1%E6%98%AF%E9%99%84%E5%8A%A0%E5%88%B0%E8%BF%9B%E7%A8%8B%E7%9A%84%E7%AA%97%E5%8F%A3.html)

经过实际的测试发现在 Win10 依然还是可以通过 RealTimeStylus 获取低延迟的实时触摸。我拿到了大尺寸屏幕平蛙厂商的高精度触摸框进行实际测试发现走 RealTimeStylus 方式比 `WM_Touch` 和 `WM_Pointer` 的延迟更低，而 `WM_Touch` 和 `WM_Pointer` 的延迟几乎相等，这个测试符合理论，我猜是对的。推荐大家自行进行测试，测试 `WM_Touch` 的 Demo 可以参阅 [WPF 编写一个测试 WM_TOUCH 触摸消息延迟的应用](https://blog.lindexi.com/post/WPF-%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AA%E6%B5%8B%E8%AF%95-WM_TOUCH-%E8%A7%A6%E6%91%B8%E6%B6%88%E6%81%AF%E5%BB%B6%E8%BF%9F%E7%9A%84%E5%BA%94%E7%94%A8.html ) 博客，测试 RealTimeStylus 的 Demo 可以参阅本文

在 WPF 框架里面，默认的触摸就是通过 WPF 的 PenImc 模块，从 RealTimeStylus 实时触摸里获取。在 WPF 里面选用 RealTimeStylus 实时触摸有两个原因，一个原因是为了更好的支持 XP 系统下的触摸，第二个是 WPF 内置了 InkCanvas 功能，默认就需要支持高性能笔迹书写。详细请看 [WPF 触摸底层 PenImc 是如何工作的](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%BA%95%E5%B1%82-PenImc-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84.html)

为什么走 RealTimeStylus 实时触摸可以较低延迟获取到触摸信息？这是因为 RealTimeStylus 的触摸数据是由 wisptis 模块提供的（Win10下暂未确定）触摸数据，在每次触摸数据收集到时，将会释放进程锁让 RealTimeStylus 层读取共享内存里的触摸数据。也就是说从触摸进到 PC 到 RealTimeStylus 层获取，这个中间隔的中间商不多



理论部分咱就先聊这里，接下来是开始从零写代码使用 RealTimeStylus 机制获取到触摸信息。通过此 Demo 不仅可以让大家了解这一套获取触摸的玩法，制作出来的 Demo 还可以让大家用来测试 RealTimeStylus 获取触摸时的延迟以及用来测试触摸失效的设备

新建一个基于 .NET 7 的 WPF 项目，新建完成之后，进入 App.xaml.cs 里将 WPF 自身的触摸模块关闭。如果没有关于 WPF 的触摸模块，那将会出现 RealTimeStylus 打架，导致自己写的 RealTimeStylus 对接代码无法工作

```csharp
        public App()
        {
            AppContext.SetSwitch("Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport", true);
        }
```

通过设置 `Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport` 即可禁用 WPF 的触摸模块，详细请参阅 [WPF 禁用实时触摸](https://blog.lindexi.com/post/WPF-%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8.html )

接下来就是一大堆无聊的 COM 对接类型定义了，本文这里只给出其中核心的关键代码，在本文末尾可以找到所有代码的下载方法

最核心的 COM 接口就是 IRealTimeStylusNative 接口了，这是我在 Microsoft.Ink 程序集里面抄的。有删减的代码如下

```csharp
[Guid("C6C77F97-545E-4873-85F2-E0FEE550B2E9")]
[SuppressUnmanagedCodeSecurity]
[InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
internal interface IRealTimeStylusNative
{
    void Enable([MarshalAs(UnmanagedType.Bool), In] bool fEnable);

    void GetHWND(out IntPtr hWnd);

    void SetHWND([In] IntPtr hWnd);

    void AddStylusSyncPlugin([In] uint iIndex, [In] IntPtr RtpSink);

    void AddStylusAsyncPlugin([In] uint iIndex, [In] IntPtr RtpQueueSink);

    void GetWindowInputRect([MarshalAs(UnmanagedType.Struct)] out PenImcRect prcWndInputRect);

    void SetWindowInputRect([MarshalAs(UnmanagedType.Struct), In] ref PenImcRect prcWndInputRect);

    void MultiTouchEnable([MarshalAs(UnmanagedType.Bool), In] bool fEnable);
}
```



按照[官方文档](https://learn.microsoft.com/en-us/windows/win32/api/rtscom/nn-rtscom-irealtimestylus)如下的建议是更加推荐使用异步的接口，而不是同步的接口。但是由于异步的接口实现相对复杂一点点，本文使用同步的接口作为例子

> We recommend that you do not use the IStylusSyncPlugin interface implementations for CPU and time-intensive operations since this blocks the packet stream flow. These operations should be conducted in IStylusAsyncPlugin interface implementation classes which run on a different thread than the thread that maintains the packet stream flow.


本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/157bc827268431501e2713fc7ac04133ba0dbf76/WhefallralajaHubeanerelair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/157bc827268431501e2713fc7ac04133ba0dbf76/WhefallralajaHubeanerelair) 欢迎访问

可以通过如下方式获取本文以上的源代码，先创建一个名为 WhefallralajaHubeanerelair 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 157bc827268431501e2713fc7ac04133ba0dbf76
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 157bc827268431501e2713fc7ac04133ba0dbf76
```

获取代码之后，进入 WhefallralajaHubeanerelair 文件夹


更多触摸相关请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html)

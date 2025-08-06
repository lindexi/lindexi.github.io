---
title: X11 使用 XSetWMNormalHints 固定窗口所在的屏幕
description: 故事的背景是我有一个带来两个屏幕的设备。我计划使用 X11 创建两个窗口，分别让这两个窗口在两个屏幕上。在不做任何处理的情况下，会受到 X 窗口管理器的影响，导致全部都在鼠标最后一次命中的屏幕上显示窗口
tags: X11
category: 
---

<!-- 发布 -->
<!-- 博客 -->

我的需求是双屏双窗口，即一个屏幕显示一个窗口。我的是 KWin 窗口管理器，默认情况下，我的正常窗口会被显示到鼠标最后一次命中的屏幕上，无论当前在 XCreateWindow 中传入的 X 和 Y 坐标是多少

本文的测试是在 UOS 上进行的，系统信息如下

```
$ cat /etc/os-release
PRETTY_NAME="UnionTech OS Desktop 20 E"
NAME="uos"
VERSION_ID="20"
VERSION="20"
ID=uos
HOME_URL="https://www.chinauos.com/"
BUG_REPORT_URL="http://bbs.chinauos.com"
VERSION_CODENAME=uranus
```

```
$ cat /etc/os-version
[Version]
SystemName=UnionTech OS Desktop
SystemName[zh_CN]=统信桌面操作系统
ProductType=Desktop
ProductType[zh_CN]=桌面
EditionName=E
EditionName[zh_CN]=E
MajorVersion=20
MinorVersion=1050
OsBuild=11068.102
```

处理器 CPU 信息如下

```
$ cat /proc/cpuinfo
processor       : 0
vendor_id       : CentaurHauls
cpu family      : 7
model           : 59
model name      : ZHAOXIN KaiXian KX-U6780A@2.7GHz
...
```

在 X 里面，通过 XDisplayWidth 和 XDisplayHeight 拿到的是虚拟屏幕的尺寸，即多个物理屏幕拼接的外接矩形虚拟范围。我的两个屏幕排放如下

<!-- ![](image/X11 使用 XSetWMNormalHints 固定窗口所在的屏幕/X11 使用 XSetWMNormalHints 固定窗口所在的屏幕0.png) -->
![](http://cdn.lindexi.site/lindexi-202585195918630.jpg)

可通过以下调用 XRRGetMonitors 的代码获取两个屏幕的信息

```csharp
// Copy from https://github.com/AvaloniaUI/Avalonia \src\Avalonia.X11\Screens\X11Screen.Providers.cs

public class Randr15ScreensImpl
{
    public Randr15ScreensImpl(nint display, nint rootWindow)
    {
        _display = display;
        var eventWindow = CreateEventWindow(display, rootWindow);
        _window = eventWindow;

        XRRSelectInput(display, _window, RandrEventMask.RRScreenChangeNotify);
    }

    public unsafe MonitorInfo[] GetMonitorInfos()
    {
        XRRMonitorInfo* monitors = XRRGetMonitors(_display, _window, true, out var count);
        var screens = new MonitorInfo[count];
        for (var c = 0; c < count; c++)
        {
            var mon = monitors[c];

            var outputs = new nint[mon.NOutput];

            for (int i = 0; i < outputs.Length; i++)
            {
                outputs[i] = mon.Outputs[i];
            }

            screens[c] = new MonitorInfo()
            {
                Name = mon.Name,
                IsPrimary = mon.Primary != 0,
                X = mon.X,
                Y = mon.Y,
                Width = mon.Width,
                Height = mon.Height,
                Outputs = outputs,
                Display = _display,
            };
        }

        return screens;
    }

    private readonly IntPtr _display;

    private readonly IntPtr _window;
}

public unsafe struct MonitorInfo
{
    public IntPtr Name;
    public bool IsPrimary;
    public int X;
    public int Y;
    public int Width;
    public int Height;
    public IntPtr[] Outputs;
    public IntPtr Display { get; init; }

    public override string ToString()
    {
        var namePtr = XGetAtomName(Display, Name);
            var name = Marshal.PtrToStringAnsi(namePtr);
        XFree(namePtr);

        return $"{name}({Name}) IsPrimary={IsPrimary} XY={X},{Y} WH={Width},{Height}";
    }
}
```

调用的方法如下

```csharp
var randr15ScreensImpl = new Randr15ScreensImpl(display, rootWindow);
var monitorInfos = randr15ScreensImpl.GetMonitorInfos();
for (var i = 0; i < monitorInfos.Length; i++)
{
    Console.WriteLine($"屏幕{i} {monitorInfos[i]}");
}

var xDisplayWidth = XDisplayWidth(display, screen);
var xDisplayHeight = XDisplayHeight(display, screen);

Console.WriteLine($"XDisplayWidth={xDisplayWidth}");
Console.WriteLine($"XDisplayHeight={xDisplayHeight}");
```

尝试运行程序，可见控制台输出如下

```
屏幕0 DisplayPort-1(343) IsPrimary=True XY=1920,309 WH=1920,1080
屏幕1 DisplayPort-0(626) IsPrimary=False XY=0,0 WH=1920,1080
XDisplayWidth=3840
XDisplayHeight=1389
```

两个屏幕都是 1920x1080 的，水平摆放，于是 XDisplayWidth 宽度就是 `1920+1920=3840` 的尺寸。高度因为存在一定的高度差，通过 `XY=1920,309` 可知道，主屏 DisplayPort-1 低了 309 大小，于是高度为 `1080+309=1389` 的尺寸。这就意味着 X 的行为上 XDisplayWidth 和 XDisplayHeight 为多个屏幕的外接矩形尺寸

和 Windows 上不同的是，在 X 上没有使用主屏当成 0,0 点坐标，意味着不会存在负数坐标系。在 X 中将最左边的显示器屏幕当成 X 坐标的 0 点，将最上方的显示器屏幕当成 Y 坐标的 0 点。这一点差异会在一些计算中坑到我，预计坑到的时候，我已经忘记我写了这篇博客

了解了基础信息，我接下来尝试为双屏创建双窗口。简单起见，我将固定写魔数，而不是真的根据屏幕而来

我将设置第 1 个窗口，显示在 0,0 坐标。设置第 2 个窗口，显示在 1920,0 坐标。预期行为就是第 1 个窗口显示在副屏 DisplayPort-0 上，第 2 个窗口显示在主屏 DisplayPort-1 上

其实现代码如下

```csharp
    public TestX11Window(int x, int y, int width, int height, nint display,nint rootWindow,int screen)
    {
        Display = display;

        XMatchVisualInfo(display, screen, 32, 4, out var info);
        var visual = info.visual;

        var valueMask =
                //SetWindowValuemask.BackPixmap
                0
                | SetWindowValuemask.BackPixel
                | SetWindowValuemask.BorderPixel
                | SetWindowValuemask.BitGravity
                | SetWindowValuemask.WinGravity
                | SetWindowValuemask.BackingStore
                | SetWindowValuemask.ColorMap
            //| SetWindowValuemask.OverrideRedirect
            ;
        var xSetWindowAttributes = new XSetWindowAttributes
        {
            backing_store = 1,
            bit_gravity = Gravity.NorthWestGravity,
            win_gravity = Gravity.NorthWestGravity,
            //override_redirect = true, // 设置窗口的override_redirect属性为True，以避免窗口管理器的干预
            colormap = XCreateColormap(display, rootWindow, visual, 0),
            border_pixel = 0,
            background_pixel = 0,
        };

        var handle = XCreateWindow(display, rootWindow, x, y, width, height, 5,
            32,
            (int) CreateWindowArgs.InputOutput,
            visual,
            (nuint) valueMask, ref xSetWindowAttributes);

        // 在 XMapWindow 之前固定在某个屏幕上
        var hints = new XSizeHints
        {
            min_width = width,
            min_height = height,
            max_width = width,
            max_height = height,

            x = x,
            y = y,
        };
        var flags = XSizeHintsFlags.PMinSize | XSizeHintsFlags.PResizeInc | XSizeHintsFlags.PPosition | XSizeHintsFlags.USPosition;
        hints.flags = (IntPtr) flags;
        XSetWMNormalHints(display, handle, ref hints);

        X11Window = handle;

        XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                                 XEventMask.PointerMotionHintMask;
        var mask = new IntPtr(0xffffff ^ (int) ignoredMask);
        XSelectInput(display, handle, mask);

        ...
    }
```

调用代码如下

```csharp
var testX11Window1 = new TestX11Window(0, 0, width, height, display, rootWindow, screen);

testX11Window1.MapWindow();
testX11Window1.Draw();

Console.WriteLine($"X11Window1={testX11Window1.X11Window}");

var testX11Window2 = new TestX11Window(1920, 0, width, height, display, rootWindow, screen);
testX11Window2.MapWindow();
testX11Window2.Draw();

Console.WriteLine($"X11Window2={testX11Window2.X11Window}");
```

以上的核心代码就是调用 XSetWMNormalHints 设置进去 XSizeHints 参数。详细请参阅 [XSetWMProperties](https://www.x.org/archive/X11R7.6/doc/man/man3/XSetWMProperties.3.xhtml )

尝试通过 XNextEvent 获取消息，可见输出控制台如下

```
屏幕0 DisplayPort-1(343) IsPrimary=True XY=1920,309 WH=1920,1080
屏幕1 DisplayPort-0(626) IsPrimary=False XY=0,0 WH=1920,1080
XDisplayWidth=3840
XDisplayHeight=1389
X11Window1=134217731
X11Window2=134217734
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=20, send_event=False, display=94542337201216, xevent=134217731, window=134217731, x=0, y=0, width=1920, height=694, border_width=0, above=134217729, override_redirect=False)
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=20, send_event=True, display=94542337201216, xevent=134217731, window=134217731, x=0, y=40, width=1920, height=694, border_width=0, above=0, override_redirect=False)
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=68, send_event=False, display=94542337201216, xevent=134217734, window=134217734, x=1920, y=0, width=1920, height=694, border_width=0, above=134217729, override_redirect=False)
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=96, send_event=True, display=94542337201216, xevent=134217734, window=134217734, x=1920, y=349, width=1920, height=694, border_width=0, above=0, override_redirect=False)
```

通过 ConfigureNotify 消息可见 window=134217734 的 X11Window2 窗口的 X 坐标确实被设置到 1920 上，且通过实际的屏幕显示内容也可以看到两个窗口被分别显示到两个显示器屏幕上

如果没有调用 XSetWMNormalHints 设置，则窗口显示过程中，收到的 ConfigureNotify 会根据鼠标最后停留在哪个屏幕上，选择对应的屏幕设置给到窗口坐标。如以下的去掉了 XSetWMNormalHints 的代码跑出来的控制台效果

```
X11Window1=134217731
X11Window2=134217734
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=19, send_event=False, display=94583148290624, xevent=134217731, window=134217731, x=0, y=0, width=1920, height=694, border_width=0, above=134217729, override_redirect=False)
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=19, send_event=True, display=94583148290624, xevent=134217731, window=134217731, x=1920, y=349, width=1920, height=694, border_width=0, above=0, override_redirect=False)
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=51, send_event=False, display=94583148290624, xevent=134217734, window=134217734, x=1920, y=0, width=1920, height=694, border_width=0, above=134217729, override_redirect=False)
ConfigureNotify XConfigureEvent (type=ConfigureNotify, serial=103, send_event=True, display=94583148290624, xevent=134217734, window=134217734, x=1920, y=655, width=1920, height=694, border_width=0, above=0, override_redirect=False)
```

此时可见 window=134217731 的 X11Window1 将和 X11Window2 一样，被设置 `x=1920` 到主屏上

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e4780e542104ca07e16735506736bb16030e47fe/X11/CeejemwhucemwaileeRerallbefe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/e4780e542104ca07e16735506736bb16030e47fe/X11/CeejemwhucemwaileeRerallbefe) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e4780e542104ca07e16735506736bb16030e47fe
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e4780e542104ca07e16735506736bb16030e47fe
```

获取代码之后，进入 X11/CeejemwhucemwaileeRerallbefe 文件夹，即可获取到源代码

更多 X11 技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

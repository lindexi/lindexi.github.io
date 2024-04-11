本文记录我从 Avalonia 框架里面学到如何创建一个全屏置顶的 X11 应用窗口的方法

<!--more-->


<!-- CreateTime:2024/04/11 07:29:55 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前，先从 Avalonia 或 [CPF](https://gitee.com/csharpui/CPF) 里面拷贝足够的代码，这部分代码可以从本文末尾找到下载方法

设置全屏的核心代码是以下三行

```csharp
ChangeWMAtoms(false, XLib.XInternAtom(display, "_NET_WM_STATE_HIDDEN", true));
ChangeWMAtoms(true, XLib.XInternAtom(display, "_NET_WM_STATE_FULLSCREEN", true));
ChangeWMAtoms(false, XLib.XInternAtom(display, "_NET_WM_STATE_MAXIMIZED_VERT", true),
    XLib.XInternAtom(display, "_NET_WM_STATE_MAXIMIZED_HORZ", true));
```

设置置顶的代码如下

```csharp
// 最顶层 类似 WPF 的 Topmost 功能
ChangeWMAtoms(true, XLib.XInternAtom(display, "_NET_WM_STATE_ABOVE", true));
```

以上代码的 ChangeWMAtoms 是一个内部方法，实现如下

```csharp
var wmState = XLib.XInternAtom(display, "_NET_WM_STATE", true);

void ChangeWMAtoms(bool enable, params IntPtr[] atoms)
{
    var xev = new XEvent
    {
        ClientMessageEvent =
        {
            type = XEventName.ClientMessage,
            send_event = true,
            window = window,
            message_type = wmState,
            format = 32,
            ptr1 = new IntPtr(enable ? 1 : 0),
            ptr2 = (IntPtr?)atoms[0] ?? IntPtr.Zero,
            ptr3 = (IntPtr?)(atoms.Length > 1 ? atoms[1] : IntPtr.Zero) ?? IntPtr.Zero,
            ptr4 = (IntPtr?)(atoms.Length > 2 ? atoms[2] : IntPtr.Zero) ?? IntPtr.Zero
        }
    };
    XLib.XSendEvent(display, rootWindow, false,
        new IntPtr((int)(EventMask.SubstructureRedirectMask | EventMask.SubstructureNotifyMask)), ref xev);
}
```

如此即可获取一个全屏且在所有窗口，包括任务栏的上层的最顶层 X11 窗口

以上代码是从 <https://github.com/AvaloniaUI/Avalonia/blob/b5db6bb0f6c19070e2a09a23231bcc1e01c40610/src/Avalonia.X11/X11Window.cs> 里面抄的

分别是 WindowState 属性的 set 方法以及 SetTopmost 方法

为了让大家能够看到窗口在最顶层的效果，接下来绘制两条线段，用来作为界面，代码如下

```csharp
var white = XLib.XWhitePixel(display, screen);
var black = XLib.XBlackPixel(display, screen);
XLib.XSetForeground(display, gc, white);
var xDisplayWidth = XLib.XDisplayWidth(display, screen);
var xDisplayHeight = XLib.XDisplayHeight(display, screen);

while (XLib.XNextEvent(display, out var xEvent) == default)
{
    if (xEvent.type == XEventName.Expose)
    {
        XLib.XDrawLine(display, window, gc, 0, 0, xDisplayWidth, xDisplayHeight);
        XLib.XDrawLine(display, window, gc, 0, xDisplayHeight, xDisplayWidth, 0);
    }
}
```

完成之后运行代码，以下是我在 Hyperv 虚拟机的运行效果，可以看到绘制的两条线段在所有应用上方，也在任务栏上方

<!-- ![](image/学习 Avalonia 框架笔记 如何创建一个全屏置顶的 X11 应用窗口/学习 Avalonia 框架笔记 如何创建一个全屏置顶的 X11 应用窗口0.png) -->
![](http://image.acmx.xyz/lindexi%2F2024410204263964.jpg)

完全的 Program.cs 文件的代码如下

```csharp
using CeaherecelallLemlalnohuce;

XLib.XInitThreads();
var display = XLib.XOpenDisplay(0);
var screen = XLib.XDefaultScreen(display);
var defaultScreen = XLib.XDefaultScreen(display);
var rootWindow = XLib.XRootWindow(display, defaultScreen);
XLib.XMatchVisualInfo(display, screen, 32, 4, out var info);
var visual = info.visual;
var valueMask = SetWindowValuemask.BackPixmap
                | SetWindowValuemask.BackPixel
                | SetWindowValuemask.BorderPixel
                | SetWindowValuemask.BitGravity
                | SetWindowValuemask.WinGravity
                | SetWindowValuemask.BackingStore
                | SetWindowValuemask.ColorMap;
var attr = new XSetWindowAttributes
{
    backing_store = 1,
    bit_gravity = Gravity.NorthWestGravity,
    win_gravity = Gravity.NorthWestGravity,
    override_redirect = false, // 参数：_overrideRedirect
    colormap = XLib.XCreateColormap(display, rootWindow, visual, 0),
};

var window = XLib.XCreateWindow(display, rootWindow, 100, 100, 320, 240, 0,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref attr);

XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                         XEventMask.PointerMotionHintMask;
var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
XLib.XSelectInput(display, window, mask);

var gc = XLib.XCreateGC(display, window, 0, 0);

XLib.XMapWindow(display, window);
XLib.XFlush(display);

#region 全屏 最顶层

var wmState = XLib.XInternAtom(display, "_NET_WM_STATE", true);

ChangeWMAtoms(false, XLib.XInternAtom(display, "_NET_WM_STATE_HIDDEN", true));
ChangeWMAtoms(true, XLib.XInternAtom(display, "_NET_WM_STATE_FULLSCREEN", true));
ChangeWMAtoms(false, XLib.XInternAtom(display, "_NET_WM_STATE_MAXIMIZED_VERT", true),
    XLib.XInternAtom(display, "_NET_WM_STATE_MAXIMIZED_HORZ", true));

// 最顶层 类似 WPF 的 Topmost 功能
ChangeWMAtoms(true, XLib.XInternAtom(display, "_NET_WM_STATE_ABOVE", true));

void ChangeWMAtoms(bool enable, params IntPtr[] atoms)
{
    var xev = new XEvent
    {
        ClientMessageEvent =
        {
            type = XEventName.ClientMessage,
            send_event = true,
            window = window,
            message_type = wmState,
            format = 32,
            ptr1 = new IntPtr(enable ? 1 : 0),
            ptr2 = (IntPtr?)atoms[0] ?? IntPtr.Zero,
            ptr3 = (IntPtr?)(atoms.Length > 1 ? atoms[1] : IntPtr.Zero) ?? IntPtr.Zero,
            ptr4 = (IntPtr?)(atoms.Length > 2 ? atoms[2] : IntPtr.Zero) ?? IntPtr.Zero
        }
    };
    XLib.XSendEvent(display, rootWindow, false,
        new IntPtr((int)(EventMask.SubstructureRedirectMask | EventMask.SubstructureNotifyMask)), ref xev);
}

#endregion

var white = XLib.XWhitePixel(display, screen);
var black = XLib.XBlackPixel(display, screen);
XLib.XSetForeground(display, gc, white);
var xDisplayWidth = XLib.XDisplayWidth(display, screen);
var xDisplayHeight = XLib.XDisplayHeight(display, screen);

while (XLib.XNextEvent(display, out var xEvent) == default)
{
    if (xEvent.type == XEventName.Expose)
    {
        XLib.XDrawLine(display, window, gc, 0, 0, xDisplayWidth, xDisplayHeight);
        XLib.XDrawLine(display, window, gc, 0, xDisplayHeight, xDisplayWidth, 0);
    }
}

XLib.XUnmapWindow(display, window);
XLib.XDestroyWindow(display, window);
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/693a137d9349bc65b5e2ed3a7c5d2480775e621a/CeaherecelallLemlalnohuce) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/693a137d9349bc65b5e2ed3a7c5d2480775e621a/CeaherecelallLemlalnohuce) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 693a137d9349bc65b5e2ed3a7c5d2480775e621a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 693a137d9349bc65b5e2ed3a7c5d2480775e621a
```

获取代码之后，进入 CeaherecelallLemlalnohuce 文件夹，即可获取到源代码

更多 Avalonia 以及 X11 等相关技术，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

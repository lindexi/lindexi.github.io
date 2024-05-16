# dotnet 后台线程设置 X11 窗口最小化

本文将告诉大家如何在 dotnet 里面的后台线程设置 X11 窗口的最小化

<!--more-->
<!-- CreateTime:2024/05/15 07:25:31 -->

<!-- 发布 -->
<!-- 博客 -->

核心设置 X11 窗口最小化的方法是 XIconifyWindow 方法，核心问题是在后台线程需要自己使用 XOpenDisplay 获取 Display 对象，且必须调用 XCloseDisplay 时才能生效

核心 C# 代码如下

```csharp
_ = Task.Run(async () =>
{
    while (true)
    {
        var display1 = XOpenDisplay(IntPtr.Zero);
        var screen1 = XDefaultScreen(display1);
        try
        {
            await Task.Delay(TimeSpan.FromSeconds(1));
            var result = XIconifyWindow(display1, windowHandle, screen1);
            Console.WriteLine($"XIconifyWindow {result}");
        }
        finally
        {
            XCloseDisplay(display1);
        }
    }
});
```

以上代码的 windowHandle 的创建代码如下

```csharp
  var display = XOpenDisplay(IntPtr.Zero);
  var screen = XDefaultScreen(display);

  ... // 忽略其他代码

  var windowHandle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
      32,
      (int)CreateWindowArgs.InputOutput,
      visual,
      (nuint)valueMask, ref xSetWindowAttributes);
```

在 Task.Run 里面，不能使用外面 display 对象，否则在 XIconifyWindow 方法将不会返回

如果没有调用 XCloseDisplay 则 XIconifyWindow 的设置是无效的

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/33dbc36c47d6f1e68265c0f0f389a566823425fd/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/33dbc36c47d6f1e68265c0f0f389a566823425fd/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 33dbc36c47d6f1e68265c0f0f389a566823425fd
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 33dbc36c47d6f1e68265c0f0f389a566823425fd
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

后续进行更多测试，找到了 XFlush 和 XSync 方法，即不需要创建和关闭 display 对象。但实际测试发现通过如下代码方式，将需要在一定的时机下才能生效，比如鼠标在窗口内晃动，或者重新激活窗口才能生效

```csharp
_ = Task.Run(async () =>
{
    while (true)
    {
        await Task.Delay(TimeSpan.FromSeconds(1));

        var result = XIconifyWindow(display, handle, screen);

        XFlush(display);
    }
});
```

完全的代码如下

```csharp
// See https://aka.ms/new-console-template for more information

using CPF.Linux;
using System;
using System.Diagnostics;
using System.Runtime;
using static CPF.Linux.XLib;

XInitThreads();
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);

var rootWindow = XDefaultRootWindow(display);

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

var xDisplayWidth = XDisplayWidth(display, screen) / 2;
var xDisplayHeight = XDisplayHeight(display, screen) / 2;
var handle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref xSetWindowAttributes);


XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                         XEventMask.PointerMotionHintMask;
var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
XSelectInput(display, handle, mask);

XMapWindow(display, handle);
XFlush(display);

var white = XWhitePixel(display, screen);
var black = XBlackPixel(display, screen);

var gc = XCreateGC(display, handle, 0, 0);
XSetForeground(display, gc, white);
XSync(display, false);

_ = Task.Run(async () =>
{
    while (true)
    {
        await Task.Delay(TimeSpan.FromSeconds(1));

        var result = XIconifyWindow(display, handle, screen);

        XFlush(display);
    }
});

while (true)
{
    var xNextEvent = XNextEvent(display, out var @event);
    if (xNextEvent != 0)
    {
        Console.WriteLine($"xNextEvent {xNextEvent}");
        break;
    }

    if (@event.type == XEventName.Expose)
    {
        XDrawLine(display, handle, gc, 0, 0, 100, 100);
    }

    Console.WriteLine(@event.type);
}

Console.WriteLine("Hello, World!");
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/cf6f4749e4f90e44b671482e289dfaf1b24f5896/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cf6f4749e4f90e44b671482e289dfaf1b24f5896/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cf6f4749e4f90e44b671482e289dfaf1b24f5896
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cf6f4749e4f90e44b671482e289dfaf1b24f5896
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

更多 X11 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
---
title: dotnet 后台线程发送 X11 窗口消息
description: 本文将告诉大家如何在 dotnet 里面的后台线程向自己进程内的窗口发送消息
tags: dotnet,X11
category: 
---

<!-- CreateTime:2024/05/15 07:25:31 -->

<!-- 发布 -->
<!-- 博客 -->

核心是通过 XSendEvent 发送消息，发送消息想要有反应需要另开 XOpenDisplay 获取 display 对象，最后再将其关闭才能发送出去

核心代码如下

```csharp
_ = Task.Run(async () =>
{
    while (true)
    {
        await Task.Delay(TimeSpan.FromSeconds(1));

        var display1 = XOpenDisplay(IntPtr.Zero);

        try
        {
            var @event = new XEvent
            {
                ClientMessageEvent =
                {
                    type = XEventName.ClientMessage,
                    send_event = true,
                    window = handle,
                    message_type = 0,
                    format = 32,
                    ptr1 = 0,
                    ptr2 = 0,
                    ptr3 = 0,
                    ptr4 = 0,
                }
            };
            XSendEvent(display1, handle, false, 0, ref @event);
        }
        finally
        {
            XCloseDisplay(display1);
        }
    }
});
```

以上的 handle 是一个 X11 窗口指针，代码如下

```csharp
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);

var handle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref xSetWindowAttributes);
```

如果在 Task.Run 后台线程里面，使用的是外面的 display 对象，则发送失败

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ee9c8da351838b0ec3b8ab577a6c9904e024517d/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ee9c8da351838b0ec3b8ab577a6c9904e024517d/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ee9c8da351838b0ec3b8ab577a6c9904e024517d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ee9c8da351838b0ec3b8ab577a6c9904e024517d
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

更多 X11 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

再经过更多的测试和阅读大佬们的示例代码，发现只需带上 XFlush 即可，更改之后的代码如下

```csharp
_ = Task.Run(async () =>
{
    while (true)
    {
        await Task.Delay(TimeSpan.FromSeconds(1));

        var @event = new XEvent
        {
            ClientMessageEvent =
            {
                type = XEventName.ClientMessage,
                send_event = true,
                window = handle,
                message_type = 0,
                format = 32,
                ptr1 = 0,
                ptr2 = 0,
                ptr3 = 0,
                ptr4 = 0,
            }
        };
        XSendEvent(display, handle, false, 0, ref @event);

        XFlush(display);
    }
});
```

这里由于需要进行多线程共用一个 display 对象，根据 X11 文档，需要添加 XInitThreads 方法才能确保安全

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

        var @event = new XEvent
        {
            ClientMessageEvent =
            {
                type = XEventName.ClientMessage,
                send_event = true,
                window = handle,
                message_type = 0,
                format = 32,
                ptr1 = 0,
                ptr2 = 0,
                ptr3 = 0,
                ptr4 = 0,
            }
        };
        XSendEvent(display, handle, false, 0, ref @event);

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

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c32c47812df8064445019dd9295867da802643ba/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c32c47812df8064445019dd9295867da802643ba/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c32c47812df8064445019dd9295867da802643ba
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c32c47812df8064445019dd9295867da802643ba
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

更多 X11 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

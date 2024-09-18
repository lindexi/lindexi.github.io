---
title: dotnet X11 窗口之间发送鼠标消息 模拟鼠标输入
description: 本文记录我阅读 Avalonia 代码过程中所学习到的在 X11 的窗口之间发送鼠标消息，可以跨进程给其他进程的窗口发送鼠标消息，通过此方式可以实现模拟鼠标输入

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/05/15 07:25:28 -->

<!-- 发布 -->
<!-- 博客 -->

直接使用 XSendEvent 给指定窗口发送消息即可，如以下示例代码

```csharp
            var xEvent = new XEvent
            {
                MotionEvent =
                {
                    type = XEventName.MotionNotify,
                    send_event = true,
                    window = Window,
                    display = Display,
                    x = x,
                    y = y
                }
            };
            XSendEvent(Display, Window, propagate: false, new IntPtr((int) (EventMask.ButtonMotionMask)), ref xEvent);
```

以上的 Window 是自己进程的主窗口，发送的相关定义代码是我从 Avalonia 和 CPF 代码仓库里面抄的，所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7636387e97780403ce473f553540a9cc1e0652ef/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7636387e97780403ce473f553540a9cc1e0652ef/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7636387e97780403ce473f553540a9cc1e0652ef
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7636387e97780403ce473f553540a9cc1e0652ef
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

以上代码是对自己进程内的主窗口发送鼠标移动消息的示例，核心代码如下

```csharp
while (true)
{
    var xNextEvent = XNextEvent(Display, out var @event);
    if (@event.type == XEventName.MotionNotify)
    {
        var x = @event.MotionEvent.x;
        var y = @event.MotionEvent.y;

        XDrawLine(Display, Window, GC, x, y, x + 100, y);
    }

    var count = XEventsQueued(Display, 0 /*QueuedAlready*/);
    if (count == 0)
    {
        for (int i = 0; i < 100; i++)
        {
            var xEvent = new XEvent
            {
                MotionEvent =
                {
                    type = XEventName.MotionNotify,
                    send_event = true,
                    window = Window,
                    display = Display,
                    x = i,
                    y = i
                }
            };
            XSendEvent(Display, Window, propagate: false, new IntPtr((int) (EventMask.ButtonMotionMask)), ref xEvent);
        }
    }
}
```

如上述代码可以看到只需更改 XSendEvent 里面的 Window 对应的参数，即可决定发送给哪个窗口。比如有两个窗口，可以通过此方式让窗口 2 收到鼠标消息时，自动转发给窗口 1 上，核心代码如下

```csharp

var handle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref xSetWindowAttributes);


var window1 = new FooWindow(handle, display);


var window2 = new FooWindow(XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
    32,
    (int) CreateWindowArgs.InputOutput,
    visual,
    (nuint) valueMask, ref xSetWindowAttributes), display);


while (true)
{
    var xNextEvent = XNextEvent(display, out var @event);
    if (xNextEvent != 0)
    {
        break;
    }

    if (@event.type == XEventName.MotionNotify)
    {
        var x = @event.MotionEvent.x;
        var y = @event.MotionEvent.y;

        if (@event.MotionEvent.window == window1.Window)
        {
            XDrawLine(display, window1.Window, window1.GC, x, y, x + 100, y);
        }
        else
        {
            var xEvent = new XEvent
            {
                MotionEvent =
                {
                    type = XEventName.MotionNotify,
                    send_event = true,
                    window = window1.Window,
                    display = display,
                    x = x,
                    y = y
                }
            };
            XSendEvent(display, window1.Window, propagate: false, new IntPtr((int)(EventMask.ButtonMotionMask)),
                ref xEvent);
        }
    }
}

class FooWindow
{
    public FooWindow(nint windowHandle, nint display)
    {
        Window = windowHandle;

        XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                                 XEventMask.PointerMotionHintMask;
        var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
        XSelectInput(display, windowHandle, mask);

        XMapWindow(display, windowHandle);
        XFlush(display);

        var screen = XDefaultScreen(display);
        var white = XWhitePixel(display, screen);

        var gc = XCreateGC(display, windowHandle, 0, 0);
        XSetForeground(display, gc, white);

        GC = gc;
    }

    public nint Window { get; }
    public IntPtr GC { get; }
}
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c8354f643998d01eed8f56757a558623e4d94a8a/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c8354f643998d01eed8f56757a558623e4d94a8a/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c8354f643998d01eed8f56757a558623e4d94a8a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c8354f643998d01eed8f56757a558623e4d94a8a
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

以上测试的是相同进程内的，跨进程其实也可以，只需要获取其他进程的窗口对应的指针即可。其实在这里我不确定 X11 的窗口 IntPtr 是否称为指针是合适的。但行为上看起来和 Windows 下的句柄非常类似

如以下的测试代码，启动自身作为新的进程，然后传入当前进程的窗口，让另一个进程获取当前进程的窗口，接着测试在另一个进程将鼠标消息发送到当前进程上

```csharp
var handle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref xSetWindowAttributes);


var window1 = new FooWindow(handle, display);
XSync(display, false);

IntPtr window2Handle = IntPtr.Zero;

if (args.Length == 0)
{
    var currentProcess = Process.GetCurrentProcess();
    var mainModuleFileName = currentProcess.MainModule!.FileName;
    Process.Start(mainModuleFileName, [window1.Window.ToString(), window1.GC.ToString()]);
}
else if (args.Length == 2)
{
    if (long.TryParse(args[0], out var otherProcessWindowHandle))
    {
        window2Handle = new IntPtr(otherProcessWindowHandle);
    }
}

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
        if (args.Length == 0)
        {
            XDrawLine(display, window1.Window, window1.GC, 0, 0, 100, 100);
        }
    }
    else if (@event.type == XEventName.MotionNotify)
    {
        var x = @event.MotionEvent.x;
        var y = @event.MotionEvent.y;

        if (window2Handle != 0 && window2GCHandle != 0)
        {
            // 绘制是无效的
            //XDrawLine(display, window2Handle, window2GCHandle, x, y, x + 100, y);

            var xEvent = new XEvent
            {
                MotionEvent =
                {
                    type = XEventName.MotionNotify,
                    send_event = true,
                    window = window2Handle,
                    display = display,
                    x = x,
                    y = y
                }
            };
            XSendEvent(display, window2Handle, propagate: false, new IntPtr((int)(EventMask.ButtonMotionMask)),
                ref xEvent);
        }
        else
        {
            XDrawLine(display, window1.Window, window1.GC, x, y, x + 100, y);
        }
    }
}
```

放在 [github](https://github.com/lindexi/lindexi_gd/tree/ec8242cfe08a0eb23ba637c655083fceb0a8edb3/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ec8242cfe08a0eb23ba637c655083fceb0a8edb3/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ec8242cfe08a0eb23ba637c655083fceb0a8edb3
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ec8242cfe08a0eb23ba637c655083fceb0a8edb3
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

通过以上测试可以发现 X11 的鼠标输入是完全可以进行模拟输入的，只需要拿到窗口指针，使用 XSendEvent 进行发送即可

再进一步的实验，也许大家也发现上面代码里面有被我注释的 XDrawLine 的调用。在 XDrawLine 里面也是传入 GC 和 Window 指针即可绘制线段，我就想着如果传入别的进程的窗口是否适合，是否就能在其他进程的窗口上绘制出内容

我尝试从另一个进程将 GC 传回来，如下面代码

```csharp
IntPtr window2Handle = IntPtr.Zero;
IntPtr window2GCHandle = IntPtr.Zero;

if (args.Length == 0)
{
    var currentProcess = Process.GetCurrentProcess();
    var mainModuleFileName = currentProcess.MainModule!.FileName;
    Process.Start(mainModuleFileName, [window1.Window.ToString(), window1.GC.ToString()]);
}
else if (args.Length == 2)
{
    if (long.TryParse(args[0], out var otherProcessWindowHandle))
    {
        window2Handle = new IntPtr(otherProcessWindowHandle);
    }

    if (long.TryParse(args[1], out var otherProcessGCHandle))
    {
        window2GCHandle = new IntPtr(otherProcessGCHandle);
    }
}

  ... // 忽略其他代码

         XDrawLine(display, window2Handle, window2GCHandle, x, y, x + 100, y);
```

此时发现运行代码，进入到 XDrawLine 报段错误，进程挂掉。原因是 gc 指针看起来是不能跨进程使用的，以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c397b872a4d2cba187e1c04f7b015c8b2ca7092c/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c397b872a4d2cba187e1c04f7b015c8b2ca7092c/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c397b872a4d2cba187e1c04f7b015c8b2ca7092c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c397b872a4d2cba187e1c04f7b015c8b2ca7092c
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

尝试自己进程创建 GC 指针，如以下核心代码

```csharp
IntPtr window2Handle = IntPtr.Zero;
IntPtr window2GCHandle = IntPtr.Zero;

if (args.Length == 0)
{
    var currentProcess = Process.GetCurrentProcess();
    var mainModuleFileName = currentProcess.MainModule!.FileName;
    Process.Start(mainModuleFileName, [window1.Window.ToString(), window1.GC.ToString()]);
}
else if (args.Length == 2)
{
    if (long.TryParse(args[0], out var otherProcessWindowHandle))
    {
        window2Handle = new IntPtr(otherProcessWindowHandle);
    }

    //if (long.TryParse(args[1], out var otherProcessGCHandle))
    //{
    //    window2GCHandle = new IntPtr(otherProcessGCHandle);
    //}
    // 不用别人传的，从窗口进行创建
    window2GCHandle = XCreateGC(display, window2Handle, 0, 0);
    Console.WriteLine($"XCreateGC Window2 {window2GCHandle}");
}
```

如此代码经过实际测试发现没有任何效果，当然了，也不会导致当前进程挂掉。以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f0cb9bd3b4e4e9184fed831bdd84ef7e4b103888/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f0cb9bd3b4e4e9184fed831bdd84ef7e4b103888/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f0cb9bd3b4e4e9184fed831bdd84ef7e4b103888
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f0cb9bd3b4e4e9184fed831bdd84ef7e4b103888
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

更多 X11 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

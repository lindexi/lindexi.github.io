在 X11 里面有和 Win32 类似的窗口之间的关系机制，如 Owner-Owned 关系，以及 Parent-Child 关系。本文将告诉大家如何进行设置以及其行为

<!--more-->


<!-- CreateTime:2024/05/17 07:23:22 -->

<!-- 发布 -->
<!-- 博客 -->

本文将大量使用到 new bing 提供的回答内容，感谢 new bing 人工智能提供的内容

## Owner-Owned 关系

- 在这种关系中，一个窗口可以被另一个窗口拥有（owner）。
- 被拥有的窗口永远显示在拥有它的那个窗口的前面。
- 当所有者窗口最小化时，它所拥有的窗口也会被隐藏。
- 当所有者窗口被销毁时，它所拥有的窗口也会被销毁。
- 当子窗口最小化时，不会影响到所有者窗口
- 子窗口可以超过所有者窗口的范围

被拥有的窗口 = 子窗口

所有者窗口 = “在拥有它的那个窗口”

即与 WPF 的 ChildWindow.Owner = MainWindow 的效果类似。以上的 ChildWindow 为子窗口，而 MainWindow 为 所有者窗口

核心 C# 代码如下

```csharp
        // 我们使用XSetTransientForHint函数将窗口a设置为窗口b的子窗口。这将确保窗口a始终在窗口b的上方
        XSetTransientForHint(Display, a, b);
```

通过关系的描述可以了解到，使用上面代码即可设置 a 窗口一定在 b 窗口上方。此方法在 XMapWindow 之前和之后调用都生效

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0331c5dd6057106df5cb179e45d34966a3eafd1b/GececurbaiduhaldiFokeejukolu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0331c5dd6057106df5cb179e45d34966a3eafd1b/GececurbaiduhaldiFokeejukolu) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0331c5dd6057106df5cb179e45d34966a3eafd1b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0331c5dd6057106df5cb179e45d34966a3eafd1b
```

获取代码之后，进入 GececurbaiduhaldiFokeejukolu 文件夹，即可获取到源代码

### 建立三个窗口关系

使用 XSetTransientForHint 时不能让一个窗口同时在两个窗口的上方，比如说有 win1 和 win2 和 win3 三个窗口，通过以下代码调用两次 XSetTransientForHint 方法，是不能让 win1 保持在 win2 和 win3 窗口的上方的

```csharp
XSetTransientForHint(display, win1, win2);
XSetTransientForHint(display, win1, win3);
```

以上代码的效果只是让 win1 保持在 win3 的上方，而断开 win1 和 win2 的关系。此时可以看到 win2 可以放在 win1 的上方，即 XSetTransientForHint 是覆盖设置的用途，而不是追加的功能

以上测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e009992625ad0020f24e25cd7625f7ad669c0218/X11/JawalwhofuYageakaje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e009992625ad0020f24e25cd7625f7ad669c0218/X11/JawalwhofuYageakaje) 上

但如果是层叠关系则是可以的，即是 win1 在 win2 之上，而 win2 在 win3 之上，这是完全没有问题的，如以下代码

```csharp
XSetTransientForHint(display, win1, win2);
XSetTransientForHint(display, win2, win3);
```

以上代码的效果就是 win3 在最底层，且 win2 保持在 win3 之上，而 win1 则在最上层

以上测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b63bd60e31eca4ca9934befdede325b829fef154/X11/JawalwhofuYageakaje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b63bd60e31eca4ca9934befdede325b829fef154/X11/JawalwhofuYageakaje) 上

### 窗口关系断开

额外说明的是 XSetTransientForHint 关系的保持也会在 XUnmapWindow 之后断开

如下面代码，设置 win1 和 win2 关系，让 win1 在 win2 的上方

```csharp
XSetTransientForHint(display, win1, win2);
```

此时如果先 XUnmapWindow 再 XMapWindow 显示 win1 窗口，那么 win1 和 win2 的关系依然保持，即依然 win1 在 win2 的上方

此时如果 XUnmapWindow 再 XMapWindow 显示 win2 窗口，那么 win1 和 win2 的关系将被断开。即此时 win1 和 win2 谁在上方取决于谁被激活，不再保持 win1 一定在 win2 的上方

以上测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a27b0bcb13d944f961a7c3e0e00140afac6494bf/X11/JawalwhofuYageakaje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a27b0bcb13d944f961a7c3e0e00140afac6494bf/X11/JawalwhofuYageakaje) 上

## Parent-Child 关系

- 在这种关系中，一个窗口是另一个窗口的父窗口。
- 子窗口只能显示在父窗口的客户区内。
- 当父窗口被隐藏时，它的所有子窗口也会被隐藏。
- 当父窗口被销毁时，它所拥有的子窗口也会被销毁。

核心 C# 代码如下

```csharp
        // 设置父子关系
        XReparentWindow(display, childWindowHandle, mainWindowHandle, 0, 0);
        XMapWindow(display, childWindowHandle);
```

需要记住在 XMapWindow 之前调用 XReparentWindow 方法，否则关系设置无效

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bcfc938d44460c3f055957910ac1082525501c29/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bcfc938d44460c3f055957910ac1082525501c29/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bcfc938d44460c3f055957910ac1082525501c29
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bcfc938d44460c3f055957910ac1082525501c29
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

建立 Parent-Child 关系之后，如果子窗口没有调用 XSelectInput 方法时，那所有在子窗口上的消息都能被所有者窗口收到，如果调用了 XSelectInput 则子窗口收到子窗口的消息，即所有者窗口被子窗口遮挡的部分将不能收到消息，被子窗口遮挡的部分的触摸或鼠标消息会被子窗口接收

简单的测试代码逻辑如下

```csharp
var xDisplayWidth = XDisplayWidth(display, screen) / 2;
var xDisplayHeight = XDisplayHeight(display, screen) / 2;
var handle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
    32,
    (int) CreateWindowArgs.InputOutput,
    visual,
    (nuint) valueMask, ref xSetWindowAttributes);


XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                         XEventMask.PointerMotionHintMask;
var mask = new IntPtr(0xffffff ^ (int) ignoredMask);
XSelectInput(display, handle, mask);

var mainWindowHandle = handle;

        // 再创建另一个窗口设置 Owner-Owned 关系
        var childWindowHandle = XCreateSimpleWindow(display, rootWindow, 0, 0, 300, 300, 5, white, black);

        XSelectInput(display, childWindowHandle, mask);

        // 设置父子关系
        XReparentWindow(display, childWindowHandle, mainWindowHandle, 50,50);
        XMapWindow(display, childWindowHandle);

while (true)
{
    var xNextEvent = XNextEvent(display, out var @event);

    if(@event.type == XEventName.MotionNotify)
    {
        if (@event.MotionEvent.window == handle)
        {
            Console.WriteLine($"Window1 {DateTime.Now:HH:mm:ss}");
        }
        else
        {
            Console.WriteLine($"Window2 {DateTime.Now:HH:mm:ss}");
        }
    }
}
```

配置了以上代码，运行项目，可以看到鼠标在子窗口上时，只能收到子窗口的消息，如下图

<!-- ![](image/dotnet 设置 X11 建立窗口之间的父子关系/dotnet 设置 X11 建立窗口之间的父子关系1.gif) -->
![](http://image.acmx.xyz/lindexi%2Fdotnet%2520%25E8%25AE%25BE%25E7%25BD%25AE%2520X11%2520%25E5%25BB%25BA%25E7%25AB%258B%25E7%25AA%2597%25E5%258F%25A3%25E4%25B9%258B%25E9%2597%25B4%25E7%259A%2584%25E7%2588%25B6%25E5%25AD%2590%25E5%2585%25B3%25E7%25B3%25BB1.gif)

以上代码有所忽略，全部的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/07fa8637c7c744935419e5a122b38718d8bc87e3/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/07fa8637c7c744935419e5a122b38718d8bc87e3/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 07fa8637c7c744935419e5a122b38718d8bc87e3
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 07fa8637c7c744935419e5a122b38718d8bc87e3
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

设置 Parent-Child 关系之后，将限制子窗口只能在主窗口的客户区范围内，即子窗口不能超过主窗口范围，如下图所示

<!-- ![](image/dotnet 设置 X11 建立窗口之间的父子关系/dotnet 设置 X11 建立窗口之间的父子关系0.png) -->
![](http://image.acmx.xyz/lindexi%2F202451610407574.jpg)

以上代码是在 XReparentWindow 方法里面设置了子窗口的坐标，让子窗口超过主窗口的范围，代码如下

```csharp
        var mainWindowHandle = handle;

        // 再创建另一个窗口设置 Owner-Owned 关系
        var childWindowHandle = XCreateSimpleWindow(display, rootWindow, 0, 0, 300, 300, 5, white, black);

        XSelectInput(display, childWindowHandle, mask);

        // 设置父子关系
        XReparentWindow(display, childWindowHandle, mainWindowHandle, 300, 50);
        XMapWindow(display, childWindowHandle);
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/fbc6151abcbeba9b54028a849f06a8796db0adf7/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/fbc6151abcbeba9b54028a849f06a8796db0adf7/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin fbc6151abcbeba9b54028a849f06a8796db0adf7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin fbc6151abcbeba9b54028a849f06a8796db0adf7
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

以下是 new bing 给出的 XReparentWindow 函数的更多信息

`XReparentWindow` 函数的作用是将一个窗口重新设置其父窗口。具体来说，如果指定的窗口已经被映射到屏幕上，`XReparentWindow` 会自动执行 `UnmapWindow` 请求，将其从当前层次结构中移除，并将其插入到指定父窗口的子级中。这个窗口会在兄弟窗口中的堆叠顺序中置于顶部。¹²

如果原始窗口已经被映射，`XReparentWindow` 还会导致 X 服务器生成一个 `ReparentNotify` 事件。在此事件中，`override_redirect` 成员被设置为窗口的相应属性。通常情况下，窗口管理器客户端应该忽略此窗口，如果此成员设置为 `True`。最后，如果原始窗口已经被映射，X 服务器会自动对其执行 `MapWindow` 请求。对于原先被遮挡的窗口，X 服务器会执行正常的曝光处理。但是，由于最终的 `MapWindow` 请求会立即遮挡初始 `UnmapWindow` 请求的某些区域，因此 X 服务器可能不会为这些区域生成 `Expose` 事件。¹

以下情况会导致 `BadMatch` 错误：
- 新的父窗口不在与旧的父窗口相同的屏幕上。
- 新的父窗口是指定窗口本身或指定窗口的下级。
- 新的父窗口是 `InputOnly` 类型，而窗口不是。
- 指定窗口具有 `ParentRelative` 背景，而新的父窗口与指定窗口的深度不同。

总之，`XReparentWindow` 允许您在 X 窗口系统中重新组织窗口的层次结构。

使用 XReparentWindow 设置的窗口关系时，子窗口将会挡住主窗口的渲染部分，即在子窗口范围内将看不到主窗口的绘制内容

其测试代码如下，先在主窗口和子窗口绘制内容

```csharp
    if (@event.type == XEventName.Expose)
    {
        if (@event.ExposeEvent.window == handle)
        {
            XDrawLine(display, handle, gc, 2, 2, xDisplayWidth - 2, xDisplayHeight - 2);
            XDrawLine(display, handle, gc, 2, xDisplayHeight - 2, xDisplayWidth - 2, 2);
        }
        else if (childWindowHandle != 0 && @event.ExposeEvent.window == childWindowHandle)
        {
            XDrawLine(display, childWindowHandle, gc, 1, 1, xDisplayWidth - 2, 1);
            XDrawLine(display, childWindowHandle, gc, 1, xDisplayHeight - 2, xDisplayWidth - 2, xDisplayHeight - 2);
            XDrawLine(display, childWindowHandle, gc, 1, 1, 1, xDisplayHeight - 2);
            XDrawLine(display, childWindowHandle, gc, xDisplayWidth - 2, xDisplayHeight - 2, xDisplayWidth - 2, xDisplayHeight - 2);
        }
    }
```

接着使用 XMoveWindow 设置子窗口坐标，此时可见子窗口所在地方将不可见主窗口绘制的内容

```csharp
    while (true)
    {
        await Task.Delay(TimeSpan.FromSeconds(1));

        await InvokeAsync(() =>
        {
            XMoveWindow(display, childWindowHandle, Random.Shared.Next(200), Random.Shared.Next(100));
        });
    }
```

全部的测试代码如下

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
    (int) CreateWindowArgs.InputOutput,
    visual,
    (nuint) valueMask, ref xSetWindowAttributes);


XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                         XEventMask.PointerMotionHintMask;
var mask = new IntPtr(0xffffff ^ (int) ignoredMask);
XSelectInput(display, handle, mask);

XMapWindow(display, handle);
XFlush(display);

var white = XWhitePixel(display, screen);
var black = XBlackPixel(display, screen);

var gc = XCreateGC(display, handle, 0, 0);
XSetForeground(display, gc, white);
XSync(display, false);

var invokeList = new List<Action>();
var invokeMessageId = new IntPtr(123123123);

async Task InvokeAsync(Action action)
{
    var taskCompletionSource = new TaskCompletionSource();
    lock (invokeList)
    {
        invokeList.Add(() =>
        {
            action();
            taskCompletionSource.SetResult();
        });
    }

    // 在 Avalonia 里面，是通过循环读取的方式，通过 XPending 判断是否有消息
    // 如果没有消息就进入自旋判断是否有业务消息和判断是否有 XPending 消息
    // 核心使用 epoll_wait 进行等待
    // 整个逻辑比较复杂
    // 这里简单处理，只通过发送 ClientMessage 的方式，告诉消息循环需要处理业务逻辑
    // 发送 ClientMessage 是一个合理的方式，根据官方文档说明，可以看到这是没有明确定义的
    // https://www.x.org/releases/X11R7.5/doc/man/man3/XClientMessageEvent.3.html
    // https://tronche.com/gui/x/xlib/events/client-communication/client-message.html
    // The X server places no interpretation on the values in the window, message_type, or data members.
    // 在 cpf 里面，和 Avalonia 实现差不多，也是在判断 XPending 是否有消息，没消息则判断是否有业务逻辑
    // 最后再进入等待逻辑。似乎 CPF 这样的方式会导致 CPU 占用略微提升
    var @event = new XEvent
    {
        ClientMessageEvent =
        {
            type = XEventName.ClientMessage,
            send_event = true,
            window = handle,
            message_type = 0,
            format = 32,
            ptr1 = invokeMessageId,
            ptr2 = 0,
            ptr3 = 0,
            ptr4 = 0,
        }
    };
    XSendEvent(display, handle, false, 0, ref @event);

    XFlush(display);

    await taskCompletionSource.Task;
}

IntPtr childWindowHandle = 0;

_ = Task.Run(async () =>
{
    await InvokeAsync(() =>
    {
        var mainWindowHandle = handle;

        // 再创建另一个窗口设置 Owner-Owned 关系
        // 创建无边框窗口
        valueMask =
            //SetWindowValuemask.BackPixmap
            0
            | SetWindowValuemask.BackPixel
            | SetWindowValuemask.BorderPixel
            | SetWindowValuemask.BitGravity
            | SetWindowValuemask.WinGravity
            | SetWindowValuemask.BackingStore
            | SetWindowValuemask.ColorMap
            | SetWindowValuemask.OverrideRedirect // [dotnet C# X11 开发笔记](https://blog.lindexi.com/post/dotnet-C-X11-%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0.html )
            ;
        xSetWindowAttributes = new XSetWindowAttributes
        {
            backing_store = 1,
            bit_gravity = Gravity.NorthWestGravity,
            win_gravity = Gravity.NorthWestGravity,
            override_redirect = true,
            colormap = XCreateColormap(display, rootWindow, visual, 0),
            border_pixel = 0,
            background_pixel = 0,
        };

        childWindowHandle = XCreateWindow(display, rootWindow, 0, 0, xDisplayWidth, xDisplayHeight, 5,
            32,
            (int) CreateWindowArgs.InputOutput,
            visual,
            (nuint) valueMask, ref xSetWindowAttributes);

        XSelectInput(display, childWindowHandle, mask);

        // 设置父子关系
        XReparentWindow(display, childWindowHandle, mainWindowHandle, 300, 50);
        XMapWindow(display, childWindowHandle);
    });

    while (true)
    {
        await Task.Delay(TimeSpan.FromSeconds(1));

        await InvokeAsync(() =>
        {
            XMoveWindow(display, childWindowHandle, Random.Shared.Next(200), Random.Shared.Next(100));
        });
    }
});

Thread.CurrentThread.Name = "主线程";

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
        if (@event.ExposeEvent.window == handle)
        {
            XDrawLine(display, handle, gc, 2, 2, xDisplayWidth - 2, xDisplayHeight - 2);
            XDrawLine(display, handle, gc, 2, xDisplayHeight - 2, xDisplayWidth - 2, 2);
        }
        else if (childWindowHandle != 0 && @event.ExposeEvent.window == childWindowHandle)
        {
            XDrawLine(display, childWindowHandle, gc, 1, 1, xDisplayWidth - 2, 1);
            XDrawLine(display, childWindowHandle, gc, 1, xDisplayHeight - 2, xDisplayWidth - 2, xDisplayHeight - 2);
            XDrawLine(display, childWindowHandle, gc, 1, 1, 1, xDisplayHeight - 2);
            XDrawLine(display, childWindowHandle, gc, xDisplayWidth - 2, xDisplayHeight - 2, xDisplayWidth - 2, xDisplayHeight - 2);
        }
    }
    else if (@event.type == XEventName.ClientMessage)
    {
        var clientMessageEvent = @event.ClientMessageEvent;
        if (clientMessageEvent.message_type == 0 && clientMessageEvent.ptr1 == invokeMessageId)
        {
            List<Action> tempList;
            lock (invokeList)
            {
                tempList = invokeList.ToList();
                invokeList.Clear();
            }

            foreach (var action in tempList)
            {
                action();
            }
        }
    }
    else if (@event.type == XEventName.MotionNotify)
    {
        if (@event.MotionEvent.window == handle)
        {
            Console.WriteLine($"Window1 {DateTime.Now:HH:mm:ss}");
        }
        else
        {
            Console.WriteLine($"Window2 {DateTime.Now:HH:mm:ss}");
        }
    }
}

Console.WriteLine("Hello, World!");
```

运行代码之后的效果如下图

<!-- ![](image/dotnet 设置 X11 建立窗口之间的父子关系/dotnet 设置 X11 建立窗口之间的父子关系2.gif) -->
![](http://image.acmx.xyz/lindexi%2Fdotnet%2520%25E8%25AE%25BE%25E7%25BD%25AE%2520X11%2520%25E5%25BB%25BA%25E7%25AB%258B%25E7%25AA%2597%25E5%258F%25A3%25E4%25B9%258B%25E9%2597%25B4%25E7%259A%2584%25E7%2588%25B6%25E5%25AD%2590%25E5%2585%25B3%25E7%25B3%25BB2.gif)

如上图，应用是透明窗口，可以看到背后的图片应用显示的内容。上述图片是使用 [WPF 基础绘图 创建和加工图片](https://blog.lindexi.com/post/WPF-%E5%9F%BA%E7%A1%80%E7%BB%98%E5%9B%BE-%E5%88%9B%E5%BB%BA%E5%92%8C%E5%8A%A0%E5%B7%A5%E5%9B%BE%E7%89%87.html ) 绘制的图片。可以看到无论是主窗口还是子窗口都能透过去。但是子窗口将会遮挡主窗口的绘制，即让子窗口直接显示窗口之后的部分内容，但不会与主窗口合成，即主窗口被子窗口挡住的部分就没有进行渲染

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bd9f8b2c8f3f42bea639677bf4ac69602b521fc0/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bd9f8b2c8f3f42bea639677bf4ac69602b521fc0/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bd9f8b2c8f3f42bea639677bf4ac69602b521fc0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bd9f8b2c8f3f42bea639677bf4ac69602b521fc0
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

更多 X11 相关，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

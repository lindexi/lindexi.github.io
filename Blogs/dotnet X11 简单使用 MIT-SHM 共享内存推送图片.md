---
title: dotnet X11 简单使用 MIT-SHM 共享内存推送图片
description: 这是我在尝试优化 Avalonia 在 Linux 上的低端设备的渲染性能时所研究的方式，本文将告诉大家如何简单使用 XShmPutImage 等 X11 的 XShm Extension 扩展方法，通过共享内存的方式推送图片
tags: dotnet,X11
category: 
---

<!-- CreateTime:2024/08/21 07:19:28 -->

<!-- 发布 -->
<!-- 博客 -->

众所周知，在 X11 里面有经典的 Client-Server 模型。客户端程序是属于 Client 角色，需要将渲染界面作为图片推送到 Server 端进行在屏幕上呈现。推送的方法可以是 XPutImage 方式，也可以是本文介绍的 X11 的 XShm Extension 的 XShmPutImage 方式

上文的 XShm 是 X Shared Memory 的缩写，核心使用的是 libc 提供的 shmget 共享内存方法作为 X 的扩展。对于 XPutImage 的优势在于，如果是 Client 和 Server 都在本机的情况下，可以减少 Client 通过 X 协议传输图片到 Server 的耗时。利用 XShmPutImage 可以实现共享内存的共享，减少传输的耗时，提升渲染性能，降低渲染延迟

相关 Avalonia 链接： <https://github.com/AvaloniaUI/Avalonia/discussions/16690#discussioncomment-10359540>

经过我在兆芯的 ZHAOXIN KaiXian KX-U6780A 的 CPU 上的实际测试，使用 XPutImage 推送界面大图能够耗时 10 多毫秒，而使用 XShmPutImage 耗时约 0 毫秒

为什么 XShmPutImage 能够如此明显提升 XPutImage 的耗时，减少推送图片的延迟？其实 XShmPutImage 里面只是做一个通知，准确来说啥都没有做。调用 XShmPutImage 时会在 XServer 端慢慢执行渲染相关逻辑
<!-- ，在下文的对 XShmPutImage 的 `send_event` 方法参数介绍时将会重新聊到这一点 -->

从 XPutImage 换成 XShmPutImage 只是减少传输的影响，对于界面渲染与合成器部分没有优化。实际减少的耗时有限，上文的实际测试的耗时影响，仅仅只是 XShmPutImage 对于 XPutImage 的耗时相同阶段上，被 XShmPutImage 给延后在其他模块了，总耗时减少上可能只有 1-2 毫秒

接下来将和大家演示如何在 X11 里面简单使用 XShm Extension 扩展方法推送图片渲染

本文使用的很多 X11 的 PInvoke 代码是从 [CPF](https://gitee.com/csharpui/CPF) 和 Avalonia 里面抄的，大家可以在本文末尾找到本文所有代码的下载方法

前置的 X11 相关知识博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

尽管在上个世纪就能找到 XShm 相关文档，但是在实际使用之前，推荐还是判断一下当前设备的 XShm 情况，判断代码如下

```csharp
    var status = XShmQueryExtension(display);
    if (status == 0)
    {
        Console.WriteLine("XShmQueryExtension failed");
    }

    status = XShmQueryVersion(display, out var major, out var minor, out var pixmaps);
    Console.WriteLine($"XShmQueryVersion: {status} major={major} minor={minor} pixmaps={pixmaps}");
```

如果以上两个函数任意一个 status 为 0 则代表失败，当前设备不能使用 XShm 扩展

在创建图片信息之前，需要先获取对应的色深的 visual 指针，本文设置尝试获取的是 32 色的，代码如下

```csharp
    XMatchVisualInfo(display, screen, depth: 32, klass: 4, out var info);
    var visual = info.visual;
```

使用 XShmCreateImage 方法创建 XImage 对象，于此同时注册 XShmSegmentInfo 信息。在 LibC 共享内存里面，共享内存的工作依赖 shmget 创建一个共享内存标识和 shmat 通过共享内存标识获取一段内存地址。这两个信息，共享内存标识和当前进程的共享内存地址信息需要存放给到 XShmSegmentInfo 信息里面，让 X 底层工作，详细请参阅 [Linux进程间通信（六）：共享内存 shmget()、shmat()、shmdt()、shmctl() - 52php - 博客园](https://www.cnblogs.com/52php/p/5861372.html )

```csharp
    const int ZPixmap = 2;
    var xShmSegmentInfo = new XShmSegmentInfo();
    var shmImage = (XImage*) XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
        (uint) width, (uint) height);
```

上面代码的 ZPixmap 格式请参阅
[dotnet 理解 X11 的 24 位或 32 位色深窗口](https://blog.lindexi.com/post/dotnet-%E7%90%86%E8%A7%A3-X11-%E7%9A%84-24-%E4%BD%8D%E6%88%96-32-%E4%BD%8D%E8%89%B2%E6%B7%B1%E7%AA%97%E5%8F%A3.html )
<!-- [dotnet 理解 X11 的 24 位或 32 位色深窗口 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18299633 ) -->

如此即可创建一个颜色深度为 32 位色深的 XImage 指针

如上文所述，使用 shmget 创建一个共享内存标识符，代码如下

```csharp
    var mapLength = width * 4 * height; // 这里的 4 表示的是一个像素使用 4 个 byte 组成，即 ARGB 一个颜色分量一个 byte 大小

    var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
```

详细关于 shmget 函数的介绍，还请参阅 [Linux进程间通信（六）：共享内存 shmget()、shmat()、shmdt()、shmctl() - 52php - 博客园](https://www.cnblogs.com/52php/p/5861372.html )

上面代码没有列举出来的 IPC_PRIVATE 和 IPC_CREAT 是两个常量，定义如下

```csharp
    // #define IPC_CREAT	01000		/* create key if key does not exist */
    // #define IPC_PRIVATE	((key_t) 0)	/* private key */

    public const int IPC_CREAT = 01000;
    public const int IPC_PRIVATE = 0;
```

更详细的代码可以在本文末尾找到本文所有代码的下载方法

以上代码获取到的 shmgetResult 局部变量就是共享内存标识，需要将其放入到 XShmSegmentInfo 的 shmid 字段里面，且依据此变量调用 Lib C 的 shmat 获取内存地址，代码如下

```csharp
    var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
    xShmSegmentInfo.shmid = shmgetResult;

    var shmaddr = shmat(shmgetResult, IntPtr.Zero, 0);
```

获取到的共享内存地址 shmaddr 需要同样也放入到 XShmSegmentInfo 的字段进行存放，也用于 XImage 的 data 指针赋值，代码如下

```csharp
    xShmSegmentInfo.shmaddr = (char*) shmaddr.ToPointer();
    shmImage->data = shmaddr;
```

以上逻辑都在 Client 端执行的，现在 Server 端还不知道信息，此时通过 XShmAttach 方法即可将其关联，让 Server 端也能知道 XImage 对应的共享内存信息，包括 shm id 共享内存标识和 shm addr 共享内存地址信息

```csharp
    XShmAttach(display, &xShmSegmentInfo);
    XFlush(display);
```

上述代码的 XFlush 非必须，只是在本演示代码里面，期望立刻将此信息推送过去而已。慢点推送不会造成问题，也不会导致延迟

通过上文方法，现在就完成了 XShm Extension 扩展的初始化逻辑。接下来即可尝试在此共享内存里面写入数据，通知给到 Server 端在界面显示即可

以下代码演示写入一个测试界面的画面到共享内存里面，代码如下，以下将绘制一个随机颜色作为纯色界面

```csharp
            // 模拟绘制界面
            var color = Random.Shared.Next();
            color = (color | 0xFF << 24); // 让 Alpha 部分是不透明的，防止显示透明色让大家以为界面看不到
            for (int i = 0; i < mapLength / 4; i++)
            {
                var p = (int*) shmaddr;
                p[i] = color;
            }
```

还请不要在意这里的设置纯色使用的是 for 循环方法，本文这里只是一个演示代码，大家开森就好

向共享内存写入数据之后，即可通过 XShmPutImage 通知 Server 端，代码如下

```csharp
            XShmPutImage(display, drawable: handle, gc, (XImage*) shmImage, src_x: 0, src_y: 0, dst_x: 0, dst_y: 0, (uint) width, (uint) height, send_event: true);

            XFlush(display);
```

此 XShmPutImage 的参数和使用方法和 XPutImage 非常相同。只是其最后一个参数 send_event 表示的是在 Server 端完成读取共享内存的数据之后，是否要发一条 Event 给到 Client 端，让 Client 端可以在 XNextEvent 读取到

为什么会需要 Server 端读取之后发送 Event 事件给到 Client 端？这是 Lib C 共享内存的一个设计问题，共享内存的读取是不带通知的，即生产端和消费端之间的写入和读取完成是没有带通知的，需要通过第三方方式进行通知。在 XShm Extension 扩展里面，生产端在 Client 端写入数据之后，通过 XShmPutImage 通知到 Server 进行消费，这也就是为什么 XShmPutImage 执行速度非常快，耗时接近 0 毫秒的原因。当 Server 端消费完成，即读取完成共享内存的数据之后，就通过发送 Event 事件给到 Client 端，让 Client 可以决定是否复用共享内存空间

如果在调用 XShmPutImage 之后，不等 Server 端回复的读取完成 XShmCompletionEvent 事件，就继续向共享内存写入数据呢？此时将会发现 Server 端读取到错误的信息，这是不合理的代码逻辑

以上就是使用 XShm Extension 扩展方法，使用共享内存发送传输界面画面图片数据

我将所有的核心代码组装起来，合起来的代码如下

```csharp
    var status = XShmQueryExtension(display);
    if (status == 0)
    {
        Console.WriteLine("XShmQueryExtension failed");
    }

    status = XShmQueryVersion(display, out var major, out var minor, out var pixmaps);
    Console.WriteLine($"XShmQueryVersion: {status} major={major} minor={minor} pixmaps={pixmaps}");

    XMatchVisualInfo(display, screen, depth: 32, klass: 4, out var info);
    var visual = info.visual;

    const int ZPixmap = 2;
    var xShmSegmentInfo = new XShmSegmentInfo();
    var shmImage = (XImage*) XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
        (uint) width, (uint) height);

    var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
    Console.WriteLine($"shmgetResult={shmgetResult:X}");

    xShmSegmentInfo.shmid = shmgetResult;

    var shmaddr = shmat(shmgetResult, IntPtr.Zero, 0);
    Console.WriteLine($"shmaddr={shmaddr:X}");

    xShmSegmentInfo.shmaddr = (char*) shmaddr.ToPointer();
    shmImage->data = shmaddr;

    XShmAttach(display, &xShmSegmentInfo);
    XFlush(display);

    var gc = XCreateGC(display, handle, 0, 0);

    // 以下为绘制界面
            // 模拟绘制界面
            var color = Random.Shared.Next();
            color = (color | 0xFF << 24);
            for (int i = 0; i < mapLength / 4; i++)
            {
                var p = (int*) shmaddr;
                p[i] = color;
            }

            // 推送图片
            XShmPutImage(display, drawable: handle, gc, (XImage*) shmImage, src_x: 0, src_y: 0, dst_x: 0, dst_y: 0, (uint) width, (uint) height, send_event: true);

            XFlush(display);
```

在实际的代码里面，推送图片 XShmPutImage 等逻辑是写到 Expose 事件里面的，完全的代码如下

```csharp
    XInitThreads();

    var display = XOpenDisplay(IntPtr.Zero);
    var screen = XDefaultScreen(display);
    var rootWindow = XDefaultRootWindow(display);

    XMatchVisualInfo(display, screen, depth: 32, klass: 4, out var info);
    var visual = info.visual;

    var xDisplayWidth = XDisplayWidth(display, screen);
    var xDisplayHeight = XDisplayHeight(display, screen);

    var width = xDisplayWidth;
    var height = xDisplayHeight;

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

    var handle = XCreateWindow(display, rootWindow, 0, 0, width, height, 5,
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

    var mapLength = width * 4 * height;
    //Console.WriteLine($"Length = {mapLength}");

    var status = XShmQueryExtension(display);
    if (status == 0)
    {
        Console.WriteLine("XShmQueryExtension failed");
    }

    status = XShmQueryVersion(display, out var major, out var minor, out var pixmaps);
    Console.WriteLine($"XShmQueryVersion: {status} major={major} minor={minor} pixmaps={pixmaps}");

    const int ZPixmap = 2;
    var xShmSegmentInfo = new XShmSegmentInfo();
    var shmImage = (XImage*) XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
        (uint) width, (uint) height);

    Console.WriteLine($"XShmCreateImage = {(IntPtr) shmImage:X} xShmSegmentInfo={xShmSegmentInfo}");

    var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
    Console.WriteLine($"shmgetResult={shmgetResult:X}");

    xShmSegmentInfo.shmid = shmgetResult;

    var shmaddr = shmat(shmgetResult, IntPtr.Zero, 0);
    Console.WriteLine($"shmaddr={shmaddr:X}");

    xShmSegmentInfo.shmaddr = (char*) shmaddr.ToPointer();
    shmImage->data = shmaddr;

    XShmAttach(display, &xShmSegmentInfo);
    XFlush(display);

    var gc = XCreateGC(display, handle, 0, 0);

    XFlush(display);

    Task.Run(() =>
    {
        var newDisplay = XOpenDisplay(IntPtr.Zero);

        while (true)
        {
            Console.ReadLine();

            var xEvent = new XEvent
            {
                ExposeEvent =
                {
                    type = XEventName.Expose,
                    send_event = true,
                    window = handle,
                    count = 1,
                    display = newDisplay,
                    x = 0,
                    y = 0,
                    width = width,
                    height = height,
                }
            };
            // [Xlib Programming Manual: Expose Events](https://tronche.com/gui/x/xlib/events/exposure/expose.html )
            XLib.XSendEvent(newDisplay, handle, propagate: false,
                new IntPtr((int) (EventMask.ExposureMask)),
                ref xEvent);

            XFlush(newDisplay);
        }

        XCloseDisplay(newDisplay);
    });

    var stopwatch = new Stopwatch();

    while (true)
    {
        var xNextEvent = XNextEvent(display, out var @event);

        if (xNextEvent != 0)
        {
            break;
        }

        if (@event.type == XEventName.Expose)
        {
            // 模拟绘制界面
            var color = Random.Shared.Next();
            color = (color | 0xFF << 24);
            for (int i = 0; i < mapLength / 4; i++)
            {
                var p = (int*) shmaddr;
                p[i] = color;
            }

            stopwatch.Restart();

            XShmPutImage(display, drawable: handle, gc, (XImage*) shmImage, src_x: 0, src_y: 0, dst_x: 0, dst_y: 0, (uint) width, (uint) height, send_event: true);

            XFlush(display);

            stopwatch.Stop();
        }
    }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2557bcf18d1aecedd57b6c8c8bc0d74ed8251977/X11/WercawchallwarnefeWhedurcachay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2557bcf18d1aecedd57b6c8c8bc0d74ed8251977/X11/WercawchallwarnefeWhedurcachay) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2557bcf18d1aecedd57b6c8c8bc0d74ed8251977
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2557bcf18d1aecedd57b6c8c8bc0d74ed8251977
```

获取代码之后，进入 X11/WercawchallwarnefeWhedurcachay 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

[XCopyArea/XPutImage which faster?](https://groups.google.com/g/comp.windows.x/c/pSy_Jztakkw )

[c++ - Perfomance of (XGetImage + XPutImage) VS XCopyArea VS (XShmGetImage + XShmPutImage) VS GTK+ - Stack Overflow](https://stackoverflow.com/questions/30200689/perfomance-of-xgetimage-xputimage-vs-xcopyarea-vs-xshmgetimage-xshmputima )

[MIT-SHM—The MIT Shared Memory Extension How the shared memory extension works](https://www.xfree86.org/current/mit-shm.html )

[善用 XShm Extension 加速貼圖](https://fred-zone.blogspot.com/2010/08/xshm-extension.html )

[c++ - How can I use XCopyArea with colors other than blue in Xlib? - Stack Overflow](https://stackoverflow.com/questions/74956364/how-can-i-use-xcopyarea-with-colors-other-than-blue-in-xlib )

[c - How to use XShmGetImage and XShmPutImage - Stack Overflow](https://stackoverflow.com/questions/43442675/how-to-use-xshmgetimage-and-xshmputimage )

[Why use XSync in X11FramebufferSurface.Blit · AvaloniaUI/Avalonia · Discussion #16690](https://github.com/AvaloniaUI/Avalonia/discussions/16690 )

[Linux进程间通信（六）：共享内存 shmget()、shmat()、shmdt()、shmctl() - 52php - 博客园](https://www.cnblogs.com/52php/p/5861372.html )

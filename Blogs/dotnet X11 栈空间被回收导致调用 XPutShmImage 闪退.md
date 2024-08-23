本文记录在使用 X11 过程中的问题，由于不正确使用导致栈空间被回收，从而在调用 XPutShmImage 时让应用闪退，此问题本质上讲只和 X11 的设计有一分钱关系，更多的问题在于我的写法上

<!--more-->


<!-- CreateTime:2024/08/23 07:17:26 -->

<!-- 发布 -->
<!-- 博客 -->

在 [上一篇博客](https://blog.lindexi.com/post/dotnet-X11-%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-MIT-SHM-%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98%E6%8E%A8%E9%80%81%E5%9B%BE%E7%89%87.html ) 里，介绍了使用 MIT-SHM 共享内存推送图片，详细请看：[dotnet X11 简单使用 MIT-SHM 共享内存推送图片](https://blog.lindexi.com/post/dotnet-X11-%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-MIT-SHM-%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98%E6%8E%A8%E9%80%81%E5%9B%BE%E7%89%87.html )
<!-- [dotnet X11 简单使用 MIT-SHM 共享内存推送图片 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18370811 ) -->

在上一篇博客里面是在顶层函数里面完成所有逻辑的，当我准备作为产品级发布时，我优化了一些代码，接着我运行程序就收到了以下错误信息

```
X Error of failed request:  BadShmSeg (invalid shared segment parameter)
  Major opcode of failed request:  130 (MIT-SHM)
  Minor opcode of failed request:  3 (X_ShmPutImage)
  Segment id in failed request:  0x0
  Serial number of failed request:  41
  Current serial number in output stream:  42
```

或者遇到了以下经典错误

```
Segmentation fault (core dumped)
```

或如下错误

```
The RX block to map as RW was not found
```

或如下错误

```
The RW block to unmap was not found
```

或如下错误

```
corrupted double-linked list
```

以上这些错误都没有很明确的错误点，但综合以上这些错误，如 上述错误的 `Segment id in failed request:  0x0` 就能说明问题，即可能在 XPutShmImage 中遇到类似野指针或指针被覆盖等问题

特别感谢 [lsj](https://blog.sdlsj.net/ ) 帮忙阅读和调试 XLib 和 XServer 的代码，帮我找到了是 XImage 里面记录的不公开的 obdata 字段指向了 0x0 的地址或其他错误的地址的问题

为了详细说明此问题的原因，我将贴出我的部分实现代码，本文的全部代码可以在本文末尾找到代码的下载方法

核心的错误实现代码如下

```csharp
    private static unsafe XShmInfo CreateXShmInfo(IntPtr display, nint visual, int width, int height, int mapLength)
    {
        ... // 忽略其他代码

        const int ZPixmap = 2;
        var xShmSegmentInfo = new XShmSegmentInfo();
        var shmImage = (XImage*)XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
            (uint)width, (uint)height);

        Console.WriteLine(
            $"XShmCreateImage = {(IntPtr)shmImage:X} xShmSegmentInfo={xShmSegmentInfo} PXShmCreateImage={new IntPtr(&xShmSegmentInfo):X}");

        var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
        Console.WriteLine($"shmgetResult={shmgetResult:X}");

        xShmSegmentInfo.shmid = shmgetResult;

        var shmaddr = shmat(shmgetResult, IntPtr.Zero, 0);
        Console.WriteLine($"shmaddr={shmaddr:X}");

        xShmSegmentInfo.shmaddr = (char*)shmaddr.ToPointer();
        shmImage->data = shmaddr;

        XShmAttach(display, &xShmSegmentInfo);
        XFlush(display);

        return new XShmInfo(shmImage, shmaddr)
        {
        };
    }
```

以上代码的 XShmInfo 的定义如下

```csharp
unsafe class XShmInfo
{
    public XShmInfo(XImage* shmImage, IntPtr shmAddr)
    {
        ShmImage = shmImage;
        ShmAddr = shmAddr;
    }

    public XImage* ShmImage { get; }

    public IntPtr ShmAddr { get; }
}
```

以上的 CreateXShmInfo 存在什么问题？核心问题在以下这句话

```csharp
        var xShmSegmentInfo = new XShmSegmentInfo();
        var shmImage = (XImage*)XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
            (uint)width, (uint)height);
```

这里的 XShmSegmentInfo 是一个结构体，结构体在此方法里的局部变量的 new 将会分配在栈上。然而栈上的内存将会在方法结束，将会弹栈，从而导致 XShmSegmentInfo 的内存地址被覆盖。调用 XShmCreateImage 时候，将 xShmSegmentInfo 局部变量的地址作为参数。这里也不能吐槽说 X11 的设计问题，只能说是咱的使用方法不正确。传入 XShmSegmentInfo 的参数，会将 XShmSegmentInfo 的地址存放到 XImage 里面记录的 obdata 字段。随着方法执行结束进行弹栈，将让 XImage 里面记录的 obdata 字段指向错误的地址，原本正确的地址空间已经被弹栈抹除，再也没有哪个地址是正确的地址的了。于是错误的 obdata 地址空间导致后续的 XShmPutImage 方法无法正确的使用共享内存，如刚好 obdata 指向的地址空间，即原 XShmSegmentInfo 的栈地址空间刚好被写入 0 的值，那将会遇到 `Segment id in failed request:  0x0` 错误。如果是一个其他的值，则可能导致指针指向飞出去，出现 Segmentation fault (core dumped) 段错误，出现 `The RW block to unmap was not found` 错误等

为了更加和大家描述这个问题，我重新根据我产品化的代码构建了一个 Demo 项目，本项目的所有代码可以在本文末尾找到下载方法

以下代码是对 [上一篇博客](https://blog.lindexi.com/post/dotnet-X11-%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-MIT-SHM-%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98%E6%8E%A8%E9%80%81%E5%9B%BE%E7%89%87.html ) 里的代码提供的封装

先定义一个 record 记录必要的渲染信息

```csharp
public record RenderInfo
(
    IntPtr Display,
    IntPtr Visual,
    int Width,
    int Height,
    int DataByteLength,
    IntPtr Handle,
    IntPtr GC
);
```

再简单创建一个 X11 窗口，代码如下

```csharp
    XInitThreads();

    var display = XOpenDisplay(IntPtr.Zero);
    var screen = XDefaultScreen(display);
    var rootWindow = XDefaultRootWindow(display);

    XMatchVisualInfo(display, screen, 32, 4, out var info);
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
        (int)CreateWindowArgs.InputOutput,
        visual,
        (nuint)valueMask, ref xSetWindowAttributes);

    XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                             XEventMask.PointerMotionHintMask;
    var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
    XSelectInput(display, handle, mask);

    XMapWindow(display, handle);
    XFlush(display);

    var gc = XCreateGC(display, handle, 0, 0);

    XFlush(display);
```

根据以上参数创建 RenderInfo 对象，代码如下

```csharp
    // 一个像素占用4个字节，于是总共的字节数就是 width * 4 * height 的长度
    var mapLength = width * 4 * height;

    var renderInfo = new RenderInfo(display, visual, width, height, mapLength, handle, gc);
```

封装 SHM 相关代码到 XShmProvider 类型，构造函数代码如下

```csharp
class XShmProvider
{
    public XShmProvider(RenderInfo renderInfo)
    {
        _renderInfo = renderInfo;
        XShmInfo = Init();
    }

    public XShmInfo XShmInfo { get; }
    private readonly RenderInfo _renderInfo;

    ... // 忽略其他代码
}
```

以上代码的 Init 是执行初始化代码，返回 XShmInfo 类型对象。在 Init 方法里面，为了更好复现问题，使用 stackalloc 抬高栈的空间。准确来说这里应该说降低栈地址空间，这是因为栈地址是向下走的，向低地址方向走的。但大概就是这个意思，大家了解就好。为什么这里想要抬高栈的空间？原因是为了让 XShmSegmentInfo 的内存地址不被后续压入方法栈的数据覆盖，而是能够被明确的覆盖，这样才能比较好复现 `Segment id in failed request:  0x0` 的情况，防止恰好读取到一个还能用但是不正确但不爆炸的地址空间，让界面没有反应但没有报错

```csharp
    private XShmInfo Init()
    {
        // 尝试抬高栈的空间
        // 用于让 XShmSegmentInfo 的内存地址不被后续压入方法栈的数据覆盖
        Span<byte> span = stackalloc byte[1024];
        Random.Shared.NextBytes(span);

        var renderInfo = _renderInfo;
        var result = CreateXShmInfo(renderInfo.Display, renderInfo.Visual, renderInfo.Width, renderInfo.Height,
            renderInfo.DataByteLength);
        return result;
    }
```

先在 Init 方法使用 stackalloc 抬高地址空间，接着再调用 CreateXShmInfo 方法，在 CreateXShmInfo 方法里面创建的 XShmSegmentInfo 结构体就在栈上距离方法压栈地址比较远的地方。如此一来，等方法执行完成之后，弹栈后，再调用新的方法，压栈，此时也难以将压入方法栈的数据覆盖到 XShmSegmentInfo 结构体的地址

以下是 CreateXShmInfo 方法的全部代码

```csharp
    private static unsafe XShmInfo CreateXShmInfo(IntPtr display, nint visual, int width, int height, int mapLength)
    {
        var status = XShmQueryExtension(display);
        if (status == 0)
        {
            throw new Exception("XShmQueryExtension failed"); // 实际使用请换成你的业务异常类型
        }

        status = XShmQueryVersion(display, out var major, out var minor, out var pixmaps);
        Console.WriteLine($"XShmQueryVersion: {status} major={major} minor={minor} pixmaps={pixmaps}");
        if (status == 0)
        {
            throw new Exception("XShmQueryVersion failed"); // 实际使用请换成你的业务异常类型
        }

        const int ZPixmap = 2;
        // 核心问题就是 XShmSegmentInfo 是结构体，在这里将在栈上分配。后续将使用栈空间的地址传递给 XShmCreateImage 方法，然而在此方法执行之后，将会弹栈，导致 XShmSegmentInfo 的内存地址被覆盖。从而让 XImage 里面记录的 obdata 字段指向错误的地址，导致后续的 XShmPutImage 方法无法正确的使用共享内存，输出如下错误
        // X Error of failed request:  BadShmSeg (invalid shared segment parameter)
        //   Major opcode of failed request:  130 (MIT-SHM)
        //   Minor opcode of failed request:  3 (X_ShmPutImage)
        //   Segment id in failed request:  0x0
        //   Serial number of failed request:  17
        //   Current serial number in output stream:  17
        // 上述错误的 `Segment id in failed request:  0x0` 就说明了问题，即 XImage 里面记录的 obdata 字段指向了 0x0 的地址。常见的错误就是类似野指针问题或者指针被覆盖的问题
        // 在本例中，我们将 XShmSegmentInfo 的在栈上分配的内存地址给到 XImage 里面记录的 obdata 字段，方法结束之后，栈空间被覆盖，导致 obdata 字段指向了错误的地址
        // 为什么刚好是 0x0 的地址呢？其实原因在于后续的 DoDraw 使用 Span<byte> span = stackalloc byte[1024 * 2]; 强行申请更多的栈空间，从而覆盖到了 XShmSegmentInfo 的内存地址。如果非 DoDraw 强行申请且保持默认为 0 的填充，则这里的错误信息 Segment id in failed request 的值会更加迷惑，甚至指向的是一个随机的地址导致 Segmentation fault (core dumped) 段错误或 The RX block to map as RW was not found 或 The RW block to unmap was not found 或 corrupted double-linked list 等
        var xShmSegmentInfo = new XShmSegmentInfo();
        var shmImage = (XImage*)XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
            (uint)width, (uint)height);

        Console.WriteLine(
            $"XShmCreateImage = {(IntPtr)shmImage:X} xShmSegmentInfo={xShmSegmentInfo} PXShmCreateImage={new IntPtr(&xShmSegmentInfo):X}");

        var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
        Console.WriteLine($"shmgetResult={shmgetResult:X}");

        xShmSegmentInfo.shmid = shmgetResult;

        var shmaddr = shmat(shmgetResult, IntPtr.Zero, 0);
        Console.WriteLine($"shmaddr={shmaddr:X}");

        xShmSegmentInfo.shmaddr = (char*)shmaddr.ToPointer();
        shmImage->data = shmaddr;

        XShmAttach(display, &xShmSegmentInfo);
        XFlush(display);

        return new XShmInfo(shmImage, shmaddr)
        {
            DebugIntPtr = new IntPtr(&xShmSegmentInfo)
        };
    }
```

以上的 XShmInfo 的 DebugIntPtr 属性是一个为了调试 xShmSegmentInfo 的地址空间而添加的调试属性。在后续可使用此属性测试获取到的地址空间的值

继续在 XShmProvider 定义 DoDraw 方法，此方法为了更好进行测试，将使用 stackalloc 申请更大的栈空间的大小，确保在 CreateXShmInfo 方法里面创建的 XShmSegmentInfo 结构体的地址空间被覆盖到，从而能够复现问题

```csharp
class XShmProvider
{
    ... // 忽略其他代码

    public unsafe void DoDraw()
    {
        // 申请两倍于压栈空间的大小，确保测试地址被覆盖到，从而能够复现问题
        Span<byte> span = stackalloc byte[1024 * 2];
        for (int i = 0; i < span.Length; i++)
        {
            span[i] = 0x00;
        }

        Console.WriteLine($"当前调试代码的内存 {*((long*)XShmInfo.DebugIntPtr):X}");

        var display = _renderInfo.Display;
        var handle = _renderInfo.Handle;
        var gc = _renderInfo.GC;
        var shmImage = XShmInfo.ShmImage;
        var width = _renderInfo.Width;
        var height = _renderInfo.Height;

        XShmPutImage(display, handle, gc, (XImage*)shmImage, 0, 0, 0, 0, (uint)width, (uint)height, true);

        XFlush(display);
    }
```

默认在 C# dotnet 里面申请的 stackalloc 空间都会执行一次清零，除非使用 AllocateUnitializedArray 或 SkipLocalsInitiAttribute 的方式，这就意味着使用 for 循环手动清零是完全多余的。只不过我这里只是简单的测试代码，我甚至将 0x00 换成 0xCC 的值进行覆盖，从而测试确保 XShmInfo.DebugIntPtr 的地址空间被覆盖

以上使用 0xCC 是为了致敬 C++ 的 烫 内存，请看 [VC中出现“烫”和“屯”的原因（栈区的每一个字节都被0xCC填充了，也就是int 3h的机器码，动态分配的堆，VC的Debug用0xCD填充堆的空间，就出现了“屯”） - findumars - 博客园](https://www.cnblogs.com/findumars/p/7128303.html )

回到测试代码，在曝光事件调用 XShmProvider 的 DoDraw 方法，代码如下

```csharp
    var xShmProvider = new XShmProvider(renderInfo);
    while (true)
    {
        var xNextEvent = XNextEvent(display, out var @event);

        if (xNextEvent != 0)
        {
            break;
        }

        if (@event.type == XEventName.Expose)
        {
            stopwatch.Restart();

            xShmProvider.DoDraw();

            stopwatch.Stop();
        }
    }
```

运行以上代码，预期将会输出以下信息

```
X Error of failed request:  BadShmSeg (invalid shared segment parameter)
  Major opcode of failed request:  130 (MIT-SHM)
  Minor opcode of failed request:  3 (X_ShmPutImage)
  Segment id in failed request:  0x0
  Serial number of failed request:  41
  Current serial number in output stream:  42
```

通过本文的分析可以看到本文所遇到的 BadShmSeg 错误和网上提供的许多机器系统环境的错误是不相同的，完全属于自己写法的问题，传入了一个栈上地址空间，最后清空栈导致地址空间记录错误信息

本文的 Program.cs 代码如下

```csharp
// See https://aka.ms/new-console-template for more information

using System.Diagnostics;
using CPF.Linux;
using static CPF.Linux.XLib;
using static CPF.Linux.XShm;
using static CPF.Linux.LibC;

unsafe
{
    XInitThreads();

    var display = XOpenDisplay(IntPtr.Zero);
    var screen = XDefaultScreen(display);
    var rootWindow = XDefaultRootWindow(display);

    XMatchVisualInfo(display, screen, 32, 4, out var info);
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
        (int)CreateWindowArgs.InputOutput,
        visual,
        (nuint)valueMask, ref xSetWindowAttributes);

    XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                             XEventMask.PointerMotionHintMask;
    var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
    XSelectInput(display, handle, mask);

    XMapWindow(display, handle);
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
                new IntPtr((int)(EventMask.ExposureMask)),
                ref xEvent);

            XFlush(newDisplay);
        }

        XCloseDisplay(newDisplay);
    });

    var stopwatch = new Stopwatch();

    // 一个像素占用4个字节，于是总共的字节数就是 width * 4 * height 的长度
    var mapLength = width * 4 * height;
    //Console.WriteLine($"Length = {mapLength}");

    var renderInfo = new RenderInfo(display, visual, width, height, mapLength, handle, gc);
    var xShmProvider = new XShmProvider(renderInfo);
    while (true)
    {
        var xNextEvent = XNextEvent(display, out var @event);

        if (xNextEvent != 0)
        {
            break;
        }

        if (@event.type == XEventName.Expose)
        {
            stopwatch.Restart();

            xShmProvider.DoDraw();

            stopwatch.Stop();
        }
        else if ((int)@event.type == 65 /*XShmCompletionEvent*/)
        {
        }
    }
}

Console.WriteLine("Hello, World!");

public record RenderInfo
(
    IntPtr Display,
    IntPtr Visual,
    int Width,
    int Height,
    int DataByteLength,
    IntPtr Handle,
    IntPtr GC
);

class XShmProvider
{
    public XShmProvider(RenderInfo renderInfo)
    {
        _renderInfo = renderInfo;
        XShmInfo = Init();
    }

    public XShmInfo XShmInfo { get; }
    private readonly RenderInfo _renderInfo;

    private XShmInfo Init()
    {
        // 尝试抬高栈的空间
        // 用于让 XShmSegmentInfo 的内存地址不被后续压入方法栈的数据覆盖
        Span<byte> span = stackalloc byte[1024];
        Random.Shared.NextBytes(span);

        var renderInfo = _renderInfo;
        var result = CreateXShmInfo(renderInfo.Display, renderInfo.Visual, renderInfo.Width, renderInfo.Height,
            renderInfo.DataByteLength);
        return result;
    }

    private static unsafe XShmInfo CreateXShmInfo(IntPtr display, nint visual, int width, int height, int mapLength)
    {
        var status = XShmQueryExtension(display);
        if (status == 0)
        {
            throw new Exception("XShmQueryExtension failed"); // 实际使用请换成你的业务异常类型
        }

        status = XShmQueryVersion(display, out var major, out var minor, out var pixmaps);
        Console.WriteLine($"XShmQueryVersion: {status} major={major} minor={minor} pixmaps={pixmaps}");
        if (status == 0)
        {
            throw new Exception("XShmQueryVersion failed"); // 实际使用请换成你的业务异常类型
        }

        const int ZPixmap = 2;
        // 核心问题就是 XShmSegmentInfo 是结构体，在这里将在栈上分配。后续将使用栈空间的地址传递给 XShmCreateImage 方法，然而在此方法执行之后，将会弹栈，导致 XShmSegmentInfo 的内存地址被覆盖。从而让 XImage 里面记录的 obdata 字段指向错误的地址，导致后续的 XShmPutImage 方法无法正确的使用共享内存，输出如下错误
        // X Error of failed request:  BadShmSeg (invalid shared segment parameter)
        //   Major opcode of failed request:  130 (MIT-SHM)
        //   Minor opcode of failed request:  3 (X_ShmPutImage)
        //   Segment id in failed request:  0x0
        //   Serial number of failed request:  17
        //   Current serial number in output stream:  17
        // 上述错误的 `Segment id in failed request:  0x0` 就说明了问题，即 XImage 里面记录的 obdata 字段指向了 0x0 的地址。常见的错误就是类似野指针问题或者指针被覆盖的问题
        // 在本例中，我们将 XShmSegmentInfo 的在栈上分配的内存地址给到 XImage 里面记录的 obdata 字段，方法结束之后，栈空间被覆盖，导致 obdata 字段指向了错误的地址
        // 为什么刚好是 0x0 的地址呢？其实原因在于后续的 DoDraw 使用 Span<byte> span = stackalloc byte[1024 * 2]; 强行申请更多的栈空间，从而覆盖到了 XShmSegmentInfo 的内存地址。如果非 DoDraw 强行申请且保持默认为 0 的填充，则这里的错误信息 Segment id in failed request 的值会更加迷惑，甚至指向的是一个随机的地址导致 Segmentation fault (core dumped) 段错误或 The RX block to map as RW was not found 或 The RW block to unmap was not found 或 corrupted double-linked list 等
        var xShmSegmentInfo = new XShmSegmentInfo();
        var shmImage = (XImage*)XShmCreateImage(display, visual, 32, ZPixmap, IntPtr.Zero, &xShmSegmentInfo,
            (uint)width, (uint)height);

        Console.WriteLine(
            $"XShmCreateImage = {(IntPtr)shmImage:X} xShmSegmentInfo={xShmSegmentInfo} PXShmCreateImage={new IntPtr(&xShmSegmentInfo):X}");

        var shmgetResult = shmget(IPC_PRIVATE, mapLength, IPC_CREAT | 0777);
        Console.WriteLine($"shmgetResult={shmgetResult:X}");

        xShmSegmentInfo.shmid = shmgetResult;

        var shmaddr = shmat(shmgetResult, IntPtr.Zero, 0);
        Console.WriteLine($"shmaddr={shmaddr:X}");

        xShmSegmentInfo.shmaddr = (char*)shmaddr.ToPointer();
        shmImage->data = shmaddr;

        XShmAttach(display, &xShmSegmentInfo);
        XFlush(display);

        return new XShmInfo(shmImage, shmaddr)
        {
            DebugIntPtr = new IntPtr(&xShmSegmentInfo)
        };
    }

    public unsafe void DoDraw()
    {
        // 申请两倍于压栈空间的大小，确保测试地址被覆盖到，从而能够复现问题
        Span<byte> span = stackalloc byte[1024 * 2];
        for (int i = 0; i < span.Length; i++)
        {
            span[i] = 0x00;
        }

        Console.WriteLine($"当前调试代码的内存 {*((long*)XShmInfo.DebugIntPtr):X}");

        var display = _renderInfo.Display;
        var handle = _renderInfo.Handle;
        var gc = _renderInfo.GC;
        var shmImage = XShmInfo.ShmImage;
        var width = _renderInfo.Width;
        var height = _renderInfo.Height;

        XShmPutImage(display, handle, gc, (XImage*)shmImage, 0, 0, 0, 0, (uint)width, (uint)height, true);

        XFlush(display);
    }
}

unsafe class XShmInfo
{
    public XShmInfo(XImage* shmImage, IntPtr shmAddr)
    {
        ShmImage = shmImage;
        ShmAddr = shmAddr;
    }

    public XImage* ShmImage { get; }

    public IntPtr ShmAddr { get; }

    public IntPtr DebugIntPtr { set; get; }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/843c229a5b590f2c1149050d0f203f53dd48f6b6/X11/DemdarhairdaKacaycheecheeleco) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/843c229a5b590f2c1149050d0f203f53dd48f6b6/X11/DemdarhairdaKacaycheecheeleco) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 843c229a5b590f2c1149050d0f203f53dd48f6b6
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 843c229a5b590f2c1149050d0f203f53dd48f6b6
```

获取代码之后，进入 X11/DemdarhairdaKacaycheecheeleco 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

参考文档：

[dotnet X11 简单使用 MIT-SHM 共享内存推送图片](https://blog.lindexi.com/post/dotnet-X11-%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-MIT-SHM-%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98%E6%8E%A8%E9%80%81%E5%9B%BE%E7%89%87.html )

[Jammy CI tests failing with: "X Error of failed request: BadShmSeg (invalid shared segment parameter)" · Issue #18726 · RobotLocomotion/drake](https://github.com/RobotLocomotion/drake/issues/18726 )

[18.04 - How to fix X Error: BadAccess, BadDrawable, BadShmSeg while running graphical application using Docker? - Ask Ubuntu](https://askubuntu.com/questions/1237400/how-to-fix-x-error-badaccess-baddrawable-badshmseg-while-running-graphical-ap )

[got 'X Error of failed request: BadShmSeg (invalid shared segment parameter)' · Issue #234 · termux/termux-x11](https://github.com/termux/termux-x11/issues/234 )

[善用 XShm Extension 加速貼圖](https://fred-zone.blogspot.com/2010/08/xshm-extension.html )

[C++ - 面向基于堆栈的缓冲区保护的 Visual C++ 支持 - Microsoft Learn](https://learn.microsoft.com/zh-cn/archive/msdn-magazine/2017/december/c-visual-c-support-for-stack-based-buffer-protection )

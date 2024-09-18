---
title: dotnet X11 多次调用 XPutImage 是否能做到渲染同步
description: 本文将告诉大家我在麒麟系统和统信系统以及分别搭配飞腾和兆芯处理器的设备上，使用连续的 XPutImage 方法推送界面，测试是否能够在一次渲染内完成。测试结论是不能做到渲染同步

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/08/24 07:07:20 -->

<!-- 发布 -->
<!-- 博客 -->

本文的核心测试代码如下

```csharp
        XPutImage(display, handle, gc, ref xImage, @event.ExposeEvent.x, @event.ExposeEvent.y, @event.ExposeEvent.x, @event.ExposeEvent.y, (uint) @event.ExposeEvent.width,
            (uint) @event.ExposeEvent.height);

        XPutImage(display, handle, gc, ref halfImage, 0, 0, halfWidth, 0, (uint) halfWidth,
            (uint) height);
```

即连续调用两次 XPutImage 方法，其中最后一次的 XPutImage 方法是让右半边覆盖掉。如果渲染能对齐同步的话，预期右半边是不会出现闪烁问题，即不会出现先在屏幕显示首个 XPutImage 绘制的全窗口图片，再显示回最后一次的 XPutImage 覆盖的画面

然而经过实际测试，窗口显示的内容将会闪烁，即连续两次 XPutImage 不能做到渲染同步，这也符合阅读 XLib 和 XServer 和 KWin 的代码了解的行为

全部的测试代码如下

```csharp
using CPF.Linux;

using SkiaSharp;

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

var xDisplayWidth = XDisplayWidth(display, screen);
var xDisplayHeight = XDisplayHeight(display, screen);

var width = xDisplayWidth;
var height = xDisplayHeight;

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

var gc = XCreateGC(display, handle, 0, 0);
var skBitmap = new SKBitmap(width, height, SKColorType.Bgra8888, SKAlphaType.Premul);
var skCanvas = new SKCanvas(skBitmap);
var xImage = CreateImage(skBitmap);

skCanvas.Clear(SKColors.Blue);
skCanvas.Flush();

var halfWidth = width / 2;
var halfSkBitmap = new SKBitmap(halfWidth, height, SKColorType.Bgra8888, SKAlphaType.Premul);
var halfSkCanvas = new SKCanvas(halfSkBitmap);
halfSkCanvas.Clear();
halfSkCanvas.Flush();
var halfImage = CreateImage(halfSkBitmap);

_ = Task.Run(() =>
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

while (true)
{
    var xNextEvent = XNextEvent(display, out var @event);

    if (xNextEvent != 0)
    {
        break;
    }

    if (@event.type == XEventName.Expose)
    {
        skCanvas.Clear(new SKColor((uint)Random.Shared.Next()).WithAlpha(0xFF));
        skCanvas.Flush();

        XPutImage(display, handle, gc, ref xImage, @event.ExposeEvent.x, @event.ExposeEvent.y, @event.ExposeEvent.x, @event.ExposeEvent.y, (uint) @event.ExposeEvent.width,
            (uint) @event.ExposeEvent.height);

        XPutImage(display, handle, gc, ref halfImage, 0, 0, halfWidth, 0, (uint) halfWidth,
            (uint) height);
        XFlush(display);
    }
}

static XImage CreateImage(SKBitmap skBitmap)
{
    const int bytePerPixelCount = 4; // RGBA 一共4个 byte 长度
    var bitPerByte = 8;

    var bitmapWidth = skBitmap.Width;
    var bitmapHeight = skBitmap.Height;

    var img = new XImage();
    int bitsPerPixel = bytePerPixelCount * bitPerByte;
    img.width = bitmapWidth;
    img.height = bitmapHeight;
    img.format = 2; //ZPixmap;
    img.data = skBitmap.GetPixels();
    img.byte_order = 0; // LSBFirst;
    img.bitmap_unit = bitsPerPixel;
    img.bitmap_bit_order = 0; // LSBFirst;
    img.bitmap_pad = bitsPerPixel;
    img.depth = bitsPerPixel;
    img.bytes_per_line = bitmapWidth * bytePerPixelCount;
    img.bits_per_pixel = bitsPerPixel;
    XInitImage(ref img);

    return img;
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b54f37030aec86fda474e99c0ad9ae941e23e1da/X11/ChihewigiKurwawhalcelfe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b54f37030aec86fda474e99c0ad9ae941e23e1da/X11/ChihewigiKurwawhalcelfe) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b54f37030aec86fda474e99c0ad9ae941e23e1da
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b54f37030aec86fda474e99c0ad9ae941e23e1da
```

获取代码之后，进入 X11/ChihewigiKurwawhalcelfe 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

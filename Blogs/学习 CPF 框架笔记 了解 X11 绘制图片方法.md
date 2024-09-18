---
title: 学习 CPF 框架笔记 了解 X11 绘制图片方法
description: 本文记录我学习 CPF 框架的笔记，本文将记录我从 CPF 框架里面学习到的如何 X11 绘制图片的方法

<!--more-->

tags: 
category: 
---

<!-- CreateTime:2024/03/29 07:20:27 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前，先感谢小红帽开源的 CPF 框架，这是一个纯 C# dotnet 实现的跨平台 UI 框架，支持Windows、Mac、Linux系统，其中 Linux 系统方面支持国产化平台，支持龙芯、飞腾、兆芯、海光等CPU平台。设计上和WPF一样的理念，任何控件都可以任意设计模板来实现各种效果
除了使用平台相关API之外，基本可以实现一次编写，到处运行。详细请参阅 <https://gitee.com/csharpui/CPF>

以下是用 AI 生成的 CPF 的宣传标语

> 这个CPF跨平台UI框架真是太棒了！不仅具有强大的跨平台兼容性，还拥有简洁直观的界面设计，让开发变得更加高效和便捷。无论是移动端还是桌面端，都能轻松实现一致的用户体验，实在是开发者的利器！强烈推荐给所有需要跨平台UI解决方案的开发团队！

在 [学习 CPF 框架笔记 了解 X11 窗口和消息基础知识](https://blog.lindexi.com/post/%E5%AD%A6%E4%B9%A0-CPF-%E6%A1%86%E6%9E%B6%E7%AC%94%E8%AE%B0-%E4%BA%86%E8%A7%A3-X11-%E7%AA%97%E5%8F%A3%E5%92%8C%E6%B6%88%E6%81%AF%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.html ) 的基础上，假定当前已创建完成了窗口，准备好了事件监听

在 X11 执行绘制图片需要在 Expose 曝光之后进行，可在 XSelectInput 里面传入监听 Expose 事件的需求，代码如下

```csharp
XSelectInput(display, window, new IntPtr((int) XEventMask.ExposureMask));
```

通过以上代码即可在 XNextEvent 方法里面收到 Expose 事件，如以下代码

```csharp
while (XNextEvent(display, out var xEvent) == default)
{
    if (xEvent.type == XEventName.Expose)
    {
        ... // 在这里绘制图片
    }
}
```


为了让大家阅读方便，以下贴出一些前置的代码

```csharp
XInitThreads();
var display = XOpenDisplay(IntPtr.Zero);
XError.Init();

var screen = XDefaultScreen(display);

var rootWindow = XDefaultRootWindow(display);

XMatchVisualInfo(display, screen, 32, 4, out var info);
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
    override_redirect = false,  // 参数：_overrideRedirect
    colormap = XCreateColormap(display, rootWindow, visual, 0),
};

var handle = XCreateWindow(display, rootWindow, 100, 100, 500, 500, 5,
    32,
    (int) CreateWindowArgs.InputOutput,
    visual,
    (nuint) valueMask, ref attr);

var window = handle;

XSelectInput(display, window, new IntPtr((int) XEventMask.ExposureMask));

XMapWindow(display, window);
XFlush(display);

var gc = XCreateGC(display, window, 0, 0);

while (XNextEvent(display, out var xEvent) == default)
{
    if (xEvent.type == XEventName.Expose)
    {
        ... // 在这里绘制图片
    }
}
```

在 X11 绘制图片可以分为两步，第一步是获取 XImage 对象，第二步是将 XImage 通过 XPutImage 方法绘制到界面

获取 XImage 对象的关键在于构建出图片的数据，在这一步本文的例子里面选择自己创建 byte 数组，通过在 byte 填充数据作为填充各个像素点的颜色。而不是读取本机的图片文件，因为读取图片文件还有一个解码的过程，解码过程和 X11 没什么关系，为了让本文示例更贴近 X11 的绘制图片，本文这里就选择自己创建图片像素 byte 数组，填充随意的数据假装是图片

自己创建 byte 数组需要先计算数组参数，在本文这里采用的是带 Alpha 通道的三原色方式，每个通道用一个 byte 表示，在 C# 里面一个 byte 是 8 个 bit 位。即一个像素为 Alpha 通道加三原色等于 4 个 byte 的大小。位图的像素数组的长度就等于长乘以宽再乘以一个像素使用多少个 byte 表示，如以下代码

```csharp
    var bitmapWidth = 50;
    var bitmapHeight = 50;

    const int bytePerPixelCount = 4;
    var bitPerByte = 8;

    var bitmapData = new byte[bitmapWidth * bitmapHeight * bytePerPixelCount];
```

如此即可创建正确的 byte 数组，接下来可以向此数组填充一些数据，假装是图片数据，如以下代码方式

```csharp
    fixed (byte* p = bitmapData)
    {
        int* pInt = (int*) p;
        var color = Random.Shared.Next();
        for (var i = 0; i < bitmapData.Length / (sizeof(int) / sizeof(byte)); i++)
        {
            *(pInt + i) = color;
        }
    }
```

以上代码采用了不安全的方式直接用 int 填充，必须说明的是上面代码仅仅只是用于随意填充颜色而已，大家可以使用自己喜欢的方式填充数组数据

由于接下来需要将图片像素 byte 数组传递给到 X11 里面，从 dotnet 的角度来讲，这属于非托管层了。根据 dotnet 的 GC 特点，对象在内存里面的指针是可变的，这将会导致如果能够直接取出 byte 数组的对象指针，且将对象指针传递给 X11 层，将可能在某次 GC 之后，图片像素 byte 数组所在内存空间变更，导致 X11 里面存放了错误的指针地址，可能造成段错误等。解决此问题的方法可以是通过不安全代码 fixed 固定对象，也可以通过 GCHandle 的方式。由于 fixed 具备语法作用块，而在绘制的业务里面，需要在图片再也不需要被使用时才能释放，也就是无法在编写代码的过程中，固定在某个时机结束 fixed 代码，因此选用 GCHandle 是一个更好的选择，如以下代码

```csharp
    GCHandle pinnedArray = GCHandle.Alloc(bitmapData, GCHandleType.Pinned);
```

通过以上代码即可告诉 CLR 层，将图片像素 byte 数组固定内存地址，即使后续发生了 GC 也不能移动当前的图片像素 byte 数组的内存空间

在正常使用里，需要在完成业务之后，调用 GCHandle 的 Free 方法进行释放固定。方便 CLR 层进行垃圾回收压缩内存空间，防止内存碎片化

```csharp
pinnedArray.Free();
```

这里需要小心一点是，需要在 X11 相关业务不再使用此图片像素数据时，才能调用 Free 方法。否则将会导致 X11 层存放一个错误的指针地址，导致内存损坏

获取到像素数组的指针，即可构建 XImage 结构体，代码如下

```csharp
    var img = new XImage();
    int bitsPerPixel = bytePerPixelCount * bitPerByte;
    img.width = bitmapWidth;
    img.height = bitmapHeight;
    img.format = 2; //ZPixmap;
    img.data = pinnedArray.AddrOfPinnedObject();
    img.byte_order = 0;// LSBFirst;
    img.bitmap_unit = bitsPerPixel;
    img.bitmap_bit_order = 0;// LSBFirst;
    img.bitmap_pad = bitsPerPixel;
    img.depth = bitsPerPixel;
    img.bytes_per_line = bitmapWidth * bytePerPixelCount;
    img.bits_per_pixel = bitsPerPixel;
```

创建完成 XImage 对象之后，需要再调用 XInitImage 进行初始化，代码如下

```csharp
    XInitImage(ref img);
```

如此即可完成第一步获取 XImage 图片对象

第二步的绘制图片只需调用 XPutImage 方法，例子代码如下

```csharp
    XPutImage(display, window, gc, ref img, srcx: 0, srcy: 0, destx: Random.Shared.Next(100), desty: Random.Shared.Next(100), (uint) img.width, (uint) img.height);
```

以上代码里面传入的 srcx 和 srcy 指的是从原图的哪里开始画起，而 destx 和 desty 则表示画到哪里

如此即可完成绘制图片逻辑

本文使用的 Program.cs 文件代码如下

```csharp
using System.Runtime.Loader;
using static CPF.Linux.XLib;
using CPF.Linux;
using System.Runtime.InteropServices;

XInitThreads();
var display = XOpenDisplay(IntPtr.Zero);
XError.Init();

var screen = XDefaultScreen(display);

var rootWindow = XDefaultRootWindow(display);

XMatchVisualInfo(display, screen, 32, 4, out var info);
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
    override_redirect = false,  // 参数：_overrideRedirect
    colormap = XCreateColormap(display, rootWindow, visual, 0),
};

var handle = XCreateWindow(display, rootWindow, 100, 100, 500, 500, 5,
    32,
    (int) CreateWindowArgs.InputOutput,
    visual,
    (nuint) valueMask, ref attr);

var window = handle;

XSelectInput(display, window, new IntPtr((int) XEventMask.ExposureMask));

XMapWindow(display, window);
XFlush(display);

var gc = XCreateGC(display, window, 0, 0);

while (XNextEvent(display, out var xEvent) == default)
{
    if (xEvent.type == XEventName.Expose)
    {
        XImage img = CreateImage();
        XPutImage(display, window, gc, ref img, 0, 0, Random.Shared.Next(100), Random.Shared.Next(100), (uint) img.width, (uint) img.height);
    }
}

unsafe XImage CreateImage()
{
    var bitmapWidth = 50;
    var bitmapHeight = 50;

    const int bytePerPixelCount = 4; // RGBA 一共4个 byte 长度
    var bitPerByte = 8;

    var bitmapData = new byte[bitmapWidth * bitmapHeight * bytePerPixelCount];

    fixed (byte* p = bitmapData)
    {
        int* pInt = (int*) p;
        var color = Random.Shared.Next();
        for (var i = 0; i < bitmapData.Length / (sizeof(int) / sizeof(byte)); i++)
        {
            *(pInt + i) = color;
        }
    }

    GCHandle pinnedArray = GCHandle.Alloc(bitmapData, GCHandleType.Pinned);

    var img = new XImage();
    int bitsPerPixel = bytePerPixelCount * bitPerByte;
    img.width = bitmapWidth;
    img.height = bitmapHeight;
    img.format = 2; //ZPixmap;
    img.data = pinnedArray.AddrOfPinnedObject();
    img.byte_order = 0;// LSBFirst;
    img.bitmap_unit = bitsPerPixel;
    img.bitmap_bit_order = 0;// LSBFirst;
    img.bitmap_pad = bitsPerPixel;
    img.depth = bitsPerPixel;
    img.bytes_per_line = bitmapWidth * bytePerPixelCount;
    img.bits_per_pixel = bitsPerPixel;
    XInitImage(ref img);

    // 除非 XImage 不再使用了，否则此时释放，将会导致 GC 之后 data 指针对应的内存不是可用的
    // 调用 XPutImage 将访问不可用内存，导致段错误，闪退
    //pinnedArray.Free();

    return img;
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b2887797a8b04676407aa45618efbbaad732a6c2/BujeeberehemnaNurgacolarje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b2887797a8b04676407aa45618efbbaad732a6c2/BujeeberehemnaNurgacolarje) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b2887797a8b04676407aa45618efbbaad732a6c2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b2887797a8b04676407aa45618efbbaad732a6c2
```

获取代码之后，进入 BujeeberehemnaNurgacolarje 文件夹，即可获取到源代码

本文将告诉大家如何在 X11 里面创建一个窗口时，设置窗口的背景颜色

<!--more-->


<!-- CreateTime:2024/06/01 07:09:31 -->

<!-- 发布 -->
<!-- 博客 -->

在 [dotnet C# 设置 X11 应用窗口背景透明](https://blog.lindexi.com/post/dotnet-C-%E8%AE%BE%E7%BD%AE-X11-%E5%BA%94%E7%94%A8%E7%AA%97%E5%8F%A3%E8%83%8C%E6%99%AF%E9%80%8F%E6%98%8E.html ) 的基础上，可以通过创建 XColor 结构体，将 XColor 赋值给到 XSetWindowAttributes 的 background_pixel 进行设置窗口的初始化背景颜色

核心实现如下

先创建 XColor 结构体，代码如下

```csharp
XColor color = new XColor()
{
    red = 0xF556, // value is 0-65535
    green = 0xC156,
    blue = 0x2156,
    flags = (byte)(ColorFlags.DoRed | ColorFlags.DoGreen | ColorFlags.DoBlue),
};
```

以上的 XColor 的三个颜色分量是 0-65535 范围的，即 0-0xFFFF 范围，而不是常见的 WPF 系的 0-0xFF 范围。换句话说如果你在 XColor 里面使用 WPF 系习惯的写法，将会发现最终颜色都是一个黑色

最后一个参数（字段）用来指明有哪些颜色分量是有效的。以上的 XColor 和 ColorFlags 类型定义都是从 CPF 和 Avalonia 里面拷贝的，可以从本文末尾找到所有的代码的下载方法

完成 XColor 创建之后，再需要使用 XCreateColormap 创建颜色映射表，代码如下

```csharp
var colormap = XCreateColormap(display, rootWindow, visual, 0);
```

以上代码的 display 和 visual 等参数的获取代码如下

```csharp
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);

var rootWindow = XDefaultRootWindow(display);

var result = XMatchVisualInfo(display, screen, 32, 4, out var info);
```

获取到所创建颜色映射表之后，即可通过 XAllocColor 获取对应的颜色了。为什么需要这一步？这是因为在X11里面支持多个不同的格式的颜色，需要经过这一步骤才能获取具体设备相关的颜色。虽然大多数时候都是咱熟悉的 RGB 的 0xAARRGGBB 格式

```csharp
XAllocColor(display, colormap, ref color);

Console.WriteLine(color.pixel.ToString("X"));
```

接着将 XColor 的 pixel 字段赋值给到 XSetWindowAttributes 的 background_pixel 字段，代码如下

```csharp
var xSetWindowAttributes = new XSetWindowAttributes
{
    backing_store = 1,
    bit_gravity = Gravity.NorthWestGravity,
    win_gravity = Gravity.NorthWestGravity,
    //override_redirect = true, // 设置窗口的override_redirect属性为True，以避免窗口管理器的干预
    colormap = colormap,
    border_pixel = 0,
    background_pixel = color.pixel,
};
```

如此执行创建窗口时，即可使用所配置的颜色。所有的代码如下

```csharp
using static CPF.Linux.XLib;
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);

var rootWindow = XDefaultRootWindow(display);

var result = XMatchVisualInfo(display, screen, 32, 4, out var info);
Console.WriteLine($"Result={result} info.depth={info.depth}");

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


var colormap = XCreateColormap(display, rootWindow, visual, 0);

XColor color = new XColor()
{
    red = 0xF556, // value is 0-65535
    green = 0xC156,
    blue = 0x2156,
    flags = (byte)(ColorFlags.DoRed | ColorFlags.DoGreen | ColorFlags.DoBlue),
};

XAllocColor(display, colormap, ref color);

Console.WriteLine(color.pixel.ToString("X"));

var xSetWindowAttributes = new XSetWindowAttributes
{
    backing_store = 1,
    bit_gravity = Gravity.NorthWestGravity,
    win_gravity = Gravity.NorthWestGravity,
    //override_redirect = true, // 设置窗口的override_redirect属性为True，以避免窗口管理器的干预
    colormap = colormap,
    border_pixel = 0,
    background_pixel = color.pixel,
};

var width = 500;
var height = 500;
var handle = XCreateWindow(display, rootWindow, 0, 0, width, height, 5,
    (int) info.depth,
    (int) CreateWindowArgs.InputOutput,
    visual,
    (nuint) valueMask, ref xSetWindowAttributes);

XMapWindow(display, handle);
XFlush(display);

while (true)
{
    var xNextEvent = XNextEvent(display, out var @event);
    if (xNextEvent != 0)
    {
        break;
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2ff0f8d8c502761d58179fa77c3928c7586ae75f/X11/YalcharlellawWaylarwejear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2ff0f8d8c502761d58179fa77c3928c7586ae75f/X11/YalcharlellawWaylarwejear) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2ff0f8d8c502761d58179fa77c3928c7586ae75f
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2ff0f8d8c502761d58179fa77c3928c7586ae75f
```

获取代码之后，进入 X11/YalcharlellawWaylarwejear 文件夹，即可获取到源代码

更多 X11 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

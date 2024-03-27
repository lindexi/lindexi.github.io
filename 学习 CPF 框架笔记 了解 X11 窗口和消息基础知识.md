# 学习 CPF 框架笔记 了解 X11 窗口和消息基础知识

本文记录我学习 CPF 框架的笔记，这是我从 CPF 框架里面学习到的 X11 的窗口和消息基础知识。本文将告诉大家如何创建一个 X11 简单窗口，支持在这个窗口上用鼠标画出简单的内容

<!--more-->
<!-- CreateTime:2024/03/26 07:13:48 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前，先感谢小红帽开源的 CPF 框架，这是一个纯 C# dotnet 实现的跨平台 UI 框架，支持Windows、Mac、Linux系统，其中 Linux 系统方面支持国产化平台，支持龙芯、飞腾、兆芯、海光等CPU平台。设计上和WPF一样的理念，任何控件都可以任意设计模板来实现各种效果
除了使用平台相关API之外，基本可以实现一次编写，到处运行。详细请参阅 <https://gitee.com/csharpui/CPF>

本文将大量参考 CPF 里面的代码，关于对 X11 的简单封装调用的代码，我不会在博客里面写出来，只放在我的示例项目代码里，可以在本文末尾找到所有代码的下载方式

以下是创建一个简单的 X11 窗口的例子

开始之前，先准备好一个空控制台项目，不需要有任何的引用库，只需要一个空的控制台项目即可。创建完成之后，开启不安全代码支持，可编辑 csproj 项目文件，替换为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>

</Project>
```

为了方便演示，接下来创建一个名为 App 的类型，这个类型没有也不需要任何的继承，只是为了方便编写代码

```csharp
class App
{
}
```

在 App 的构造函数里，先调用 XInitThreads 方法进行初始化线程，如以下代码

```csharp
class App
{
    public App()
    {
        XInitThreads();

        ... // 忽略其他代码
    }
}
```

以上代码的 XInitThreads 方法是对 X11 的一个简单的封装，其代码定义如下

```csharp
namespace CPF.Linux
{
    public unsafe static class XLib
    {
        const string libX11 = "libX11.so.6";

        [DllImport(libX11)]
        public static extern int XInitThreads();

        ... // 忽略其他代码
    }
}
```

为了能够找 App 类型里面更简单的调用 XInitThreads 方法，在 App 里面引用静态类，如以下代码

```csharp
using System.Runtime.Loader;
using static CPF.Linux.XLib;
using CPF.Linux;

class App
{
    public App()
    {
        XInitThreads();

        ... // 忽略其他代码
    }
}
```

如此即可方便的调用到 XInitThreads 方法了

为了方便调试，定义名为 XError 的类型，在这个类型里面负责使用 XSetErrorHandler 注册错误处理

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace CPF.Linux
{
    static class XError
    {
        private static readonly XErrorHandler s_errorHandlerDelegate = Handler;
        public static XErrorEvent LastError;
        static int Handler(IntPtr display, ref XErrorEvent error)
        {
            LastError = error;
            //StringBuilder stringBuilder = new StringBuilder(100);
            //XLib.XGetErrorText(error.display, error.error_code, stringBuilder, stringBuilder.Length);
            //Console.WriteLine("异常:" + stringBuilder.ToString() + " " + error.request_code + ":" + error.error_code);
            return 0;
        }

        public static void ThrowLastError(string desc)
        {
            var err = LastError;
            LastError = new XErrorEvent();
            if (err.error_code == 0)
                throw new Exception(desc);
            throw new Exception(desc + ": " + err.error_code);

        }

        public static void Init()
        {
            XLib.XSetErrorHandler(s_errorHandlerDelegate);
        }
    }
}
```

日常调试可以在 XError 的 Handler 方法里面打上断点，这样如果出现异常了，就可以立刻拿到

完成 XError 的定义之后，即可在 App 构造函数里使用

```csharp
    public App()
    {
        XInitThreads();
        XError.Init();

        ... // 忽略其他代码
    }
```

打开或获取 Display 对象，以及获取屏幕，代码如下

```csharp
class App
{
    public App()
    {
        XInitThreads();
        XError.Init();
        Display = XOpenDisplay(IntPtr.Zero);
        var screen = XDefaultScreen(Display);
        Screen = screen;

        Info = new X11Info(Display, DeferredDisplay);
        ... // 忽略其他代码
    }
    public IntPtr DeferredDisplay { get; set; }
    public IntPtr Display { get; set; }
    public int Screen { get; set; }
    public X11Info Info { get; private set; }

    ... // 忽略其他代码
}
```

以上代码的 X11Info 是一个自定义的类型，在本文这里只用来存放 Display 对象而已，没有其他作用。只是在 CPF 里面会用来表示 X11 的状态，比如输入版本等，于是在本文这里就继续抄 CPF 的实现逻辑。但在本文演示逻辑里面，只用 Info 对象用来存放 Display 对象而已

完成以上的获取屏幕等信息之后，就可以来创建窗口。创建窗口需要额外创建前景色和背景色，使用如下代码创建白色和黑色

```csharp
        var white = XWhitePixel(Display, screen);
        var black = XBlackPixel(Display, screen);
```

获取到两个颜色之后，分别作为边框前景色和窗口背景色，使用 XCreateSimpleWindow 创建窗口，代码如下


```csharp
class App
{
    public App()
    {
        XInitThreads();
        XError.Init();
        Display = XOpenDisplay(IntPtr.Zero);
        var screen = XDefaultScreen(Display);
        Screen = screen;

        Info = new X11Info(Display, DeferredDisplay);

        var white = XWhitePixel(Display, screen);
        var black = XBlackPixel(Display, screen);
        Window = XCreateSimpleWindow(Display, XDefaultRootWindow(Display), 0, 0, 300, 300, 5, white, black);
        ... // 忽略其他代码
    }

    ... // 忽略其他代码
}
```

通过以上代码即可完成窗口的创建，但创建完成的窗口还没显示出来。不同于 WPF 的 Show 方法，在 X11 里面需要先将窗口关联屏幕，然后再使用 XRaiseWindow 或 XMapRaised 显示窗口，代码如下

```csharp
        XMapWindow(Display, Window);
        XFlush(Info.Display);

        XRaiseWindow(Display, Window); // 可选。理论上只需使用 XMapWindow 方法即可在屏幕里显示出窗口
```

以上的代码就是最简单的显示窗口的代码。 为了能够在窗口里面绘制内容以及接收输入，还需要添加更多额外的代码。如使用 XSelectInput 方法配置此窗口接收哪些输入。如果没有调用 XSelectInput 方法，那在后续的 XNextEvent 将无法收到任何的输入消息。以下代码是先设置有哪些消息是忽略的，再使用 `0xffffff ^ (int)ignoredMask` 即可获取到所有的不在忽略列表里面的事件

```csharp
        XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                                 XEventMask.PointerMotionHintMask;
        var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
        XSelectInput(Display, Window, mask);
```

使用 XCreateGC 方法可以获取到用于绘图的指针，代码如下。后续可用 GC 属性辅助绘制界面内容

```csharp
class App
{
    public App()
    {
        GC = XCreateGC(Display, Window, 0, 0);
        ... // 忽略其他代码
    }

    ... // 忽略其他代码
    private IntPtr GC { get; }
}
```

完成以上修改的 App 构造函数的代码如下

```csharp
    public App()
    {
        XInitThreads();
        Display = XOpenDisplay(IntPtr.Zero);
        XError.Init();
        Info = new X11Info(Display, DeferredDisplay);
        Console.WriteLine("XInputVersion=" + Info.XInputVersion);
        var screen = XDefaultScreen(Display);
        Console.WriteLine($"Screen = {screen}");
        Screen = screen;
        var white = XWhitePixel(Display, screen);
        var black = XBlackPixel(Display, screen);
        Window = XCreateSimpleWindow(Display, XDefaultRootWindow(Display), 0, 0, 300, 300, 5, white, black);

        Console.WriteLine($"Window={Window}");

        XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                                 XEventMask.PointerMotionHintMask;
        var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
        XSelectInput(Display, Window, mask);

        XMapWindow(Display, Window);

        XFlush(Info.Display);
        GC = XCreateGC(Display, Window, 0, 0);

        XSetForeground(Display, GC, white);
    }
```

按照 CPF 或 WPF 等的设计，将会在 App 类型里面添加一个 Run 方法，在这个方法里面循环读取输入，代码如下

```csharp
class App
{
    public void Run()
    {
        ... // 忽略其他代码
    }

    ... // 忽略其他代码
}
```

先在 Run 方法里面调用 XRaiseWindow 显示窗口，再调用 XSetInputFocus 获取焦点，代码如下

```csharp
class App
{
    public void Run()
    {
        XRaiseWindow(Display, Window);
        XSetInputFocus(Display, Window, 0, IntPtr.Zero);

        ... // 忽略其他代码
    }

    ... // 忽略其他代码
}
```

接着进入一个无限循环里，在里面不断获取 XNextEvent 事件，代码如下

```csharp
class App
{
    public void Run()
    {
        XRaiseWindow(Display, Window);
        XSetInputFocus(Display, Window, 0, IntPtr.Zero);

        while (true)
        {
            XSync(Display, false);

            var xNextEvent = XNextEvent(Display, out var @event);

             ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }

    ... // 忽略其他代码
}
```

使用以上代码即可获取到输入事件，接下来咱将使用输入的鼠标事件尝试绘制内容。绘制内容的开始需要先设置绘制的内容的前景色，比如绘制的线段的前景色，代码如下

```csharp
        XSetForeground(Display, GC, white);
```

在 X11 里面的类似于 Win32 的 WM_Paint 消息的是一个名为 Expose 的曝光的事件，当从 XNextEvent 收到此事件时，可以执行重绘逻辑。但值得一提的是，在 X11 里面，可以在任意的逻辑里面执行绘图，而不是只能在曝光事件里面执行。这就意味着在本文的例子里面，不需要在曝光事件里面执行任何逻辑

```csharp
class App
{
    public void Run()
    {
        XRaiseWindow(Display, Window);
        XSetInputFocus(Display, Window, 0, IntPtr.Zero);

        while (true)
        {
            XSync(Display, false);

            var xNextEvent = XNextEvent(Display, out var @event);


            if (@event.type == XEventName.Expose)
            {
                Redraw();
            }
             ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }

    private void Redraw()
    {
    }

    ... // 忽略其他代码
}
```

本文例子里面将在鼠标按下拖动的过程中，绘制拖动的线，通过和 WPF 的 MouseDown 和 MouseMove 和 MouseUp 分别对应的 ButtonPress 和 MotionNotify 和 ButtonRelease 即可获取到鼠标按下拖动

```csharp
class App
{
    public void Run()
    {
        XRaiseWindow(Display, Window);
        XSetInputFocus(Display, Window, 0, IntPtr.Zero);

        while (true)
        {
            XSync(Display, false);

            var xNextEvent = XNextEvent(Display, out var @event);


            if (@event.type == XEventName.Expose)
            {
                Redraw();
            }
            else if (@event.type == XEventName.ButtonPress)
            {
                _lastPoint = (@event.ButtonEvent.x, @event.ButtonEvent.y);
                _isDown = true;
            }
            else if (@event.type == XEventName.MotionNotify)
            {
                if (_isDown)
                {
                    XDrawLine(Display, Window, GC, _lastPoint.X, _lastPoint.Y, @event.MotionEvent.x,
                        @event.MotionEvent.y);
                    _lastPoint = (@event.MotionEvent.x, @event.MotionEvent.y);
                }
            }
            else if (@event.type == XEventName.ButtonRelease)
            {
                _isDown = false;
            }
             ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }

    private (int X, int Y) _lastPoint;
    private bool _isDown;

    private void Redraw()
    {
    }

    ... // 忽略其他代码
}
```

使用以上代码之后，即可在鼠标按下拖动的过程中，在 XDrawLine 方法里面绘制出鼠标拖动的线

只是以上代码绘制的线十分粗糙

完成之后的 App 类型的代码如下

```csharp
class App
{
    public App()
    {
        XInitThreads();
        Display = XOpenDisplay(IntPtr.Zero);
        XError.Init();
        Info = new X11Info(Display, DeferredDisplay);
        Console.WriteLine("XInputVersion=" + Info.XInputVersion);
        var screen = XDefaultScreen(Display);
        Console.WriteLine($"Screen = {screen}");
        Screen = screen;
        var white = XWhitePixel(Display, screen);
        var black = XBlackPixel(Display, screen);
        Window = XCreateSimpleWindow(Display, XDefaultRootWindow(Display), 0, 0, 300, 300, 5, white, black);

        Console.WriteLine($"Window={Window}");

        XEventMask ignoredMask = XEventMask.SubstructureRedirectMask | XEventMask.ResizeRedirectMask |
                                 XEventMask.PointerMotionHintMask;
        var mask = new IntPtr(0xffffff ^ (int)ignoredMask);
        XSelectInput(Display, Window, mask);

        XMapWindow(Display, Window);

        XFlush(Info.Display);
        GC = XCreateGC(Display, Window, 0, 0);

        XSetForeground(Display, GC, white);
    }

    public void Run()
    {
        XRaiseWindow(Display, Window);
        XSetInputFocus(Display, Window, 0, IntPtr.Zero);

        while (true)
        {
            XSync(Display, false);

            var xNextEvent = XNextEvent(Display, out var @event);
            //Console.WriteLine($"NextEvent={xNextEvent} {@event}");

            if (@event.type == XEventName.Expose)
            {
                Redraw();
            }
            else if (@event.type == XEventName.ButtonPress)
            {
                _lastPoint = (@event.ButtonEvent.x, @event.ButtonEvent.y);
                _isDown = true;
            }
            else if (@event.type == XEventName.MotionNotify)
            {
                if (_isDown)
                {
                    XDrawLine(Display, Window, GC, _lastPoint.X, _lastPoint.Y, @event.MotionEvent.x,
                        @event.MotionEvent.y);
                    _lastPoint = (@event.MotionEvent.x, @event.MotionEvent.y);
                }
            }
            else if (@event.type == XEventName.ButtonRelease)
            {
                _isDown = false;
            }

            if (xNextEvent != 0)
            {
                break;
            }
        }
    }

    private (int X, int Y) _lastPoint;
    private bool _isDown;

    private void Redraw()
    {
    }

    private IntPtr GC { get; }

    public IntPtr DeferredDisplay { get; set; }
    public IntPtr Display { get; set; }

    //public XI2Manager XI2;
    public X11Info Info { get; private set; }
    public IntPtr Window { get; set; }
    public int Screen { get; set; }
}
```

可在 Program 类型里面，使用以下代码运行

```csharp
internal class Program
{
    [STAThread]
    static void Main(string[] args)
    {
        StartX11App();
    }

    private static void StartX11App()
    {
        var app = new App();
        app.Run();
    }
}
```

我尝试在 UOS 系统搭配兆芯的 CPU 的机器上，进行测试，发现绘制的延迟十分低。更具体的 UOS 内核版本号是 4.19-amd64-desktop 版本，处理器是 ZHAOXIN KaiXian KX-U6780A 型号

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/41848e97e1ea7d86c548577681a234d863cd49a1/BujeeberehemnaNurgacolarje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/41848e97e1ea7d86c548577681a234d863cd49a1/BujeeberehemnaNurgacolarje) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 41848e97e1ea7d86c548577681a234d863cd49a1
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 41848e97e1ea7d86c548577681a234d863cd49a1
```

获取代码之后，进入 BujeeberehemnaNurgacolarje 文件夹，即可获取到源代码
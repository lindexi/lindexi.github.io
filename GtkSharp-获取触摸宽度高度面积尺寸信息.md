
# GtkSharp 获取触摸宽度高度面积尺寸信息

本文将告诉大家如何在 C# dotnet 里面，从 GTK 里面获取到触摸的宽度高度信息，即触摸面积或触摸尺寸信息

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

准确来说本文的方法是我在 [lsj](https://blog.sdlsj.net) 的帮助下试出来的，我没有找到正式的文档对此有描述。本文测试的机器是在 UOS 系统搭配兆芯的 CPU 的机器上，这台机器采用的是希沃的某款触摸框的设备。更具体的 UOS 内核版本号是 4.19-amd64-desktop 版本，处理器是 ZHAOXIN KaiXian KX-U6780A 型号

核心获取方法是通过在 EventTouch 的 Axes 里面读取值。这里的 EventTouch 对应 GTK 官方文档记录的 <https://docs.gtk.org/gdk3/struct.EventTouch.html>

尽管在 [GTK 官方文档](https://docs.gtk.org/gdk3/struct.EventTouch.html) 对 axes 的描述似乎不是这个意思，以下是官方原文

> x, y translated to the axes of device, or NULL if device is the mouse.

从官方的定义上可以看到 axes 是一个 double 类型的数组，我就从数组里面的第 3、4 项分别获取到 [0-1] 范围内的宽度和高度的值。从网上一些不权威的文档里面，我看到了第 3、4 项表示的是 Xtilt 和 Ytilt 的含义，即 X 倾斜和 Y 倾斜的含义。但从实际测试上看，这个值却能够和触摸的宽度和高度对应

接下来我将使用一个简单的项目告诉大家具体如何获取触摸宽度高度信息

先新建一个控制台项目，然后编辑 csproj 项目文件，替换为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="GtkSharp" Version="3.24.24.95" />
  </ItemGroup>

</Project>
```

以上代码安装了 3.24.24.95 版本的 GtkSharp 库，同时开启不安全代码

由于当前 GtkSharp 的官方定义里面，没有开放 EventTouch 的 Axes 属性，于是咱先自己定义一个 EventTouch 结构体。为什么自己定义的结构体也能生效？这是因为官方定义的结构体的作用仅仅只是在 Marshal.PtrToStructure 从指针进行转换的，于是自己定义的结构体也能通过相同的方式从指针进行转换，代码如下

```csharp
[StructLayout(LayoutKind.Sequential)]
public partial struct EventTouch
{
    public Gdk.EventType Type;
    private IntPtr _window;
    public Gdk.Window Window
    {
        get
        {
            return GLib.Object.GetObject(_window) as Gdk.Window;
        }
        set
        {
            _window = value == null ? IntPtr.Zero : value.Handle;
        }
    }
    public sbyte SendEvent;
    public uint Time;
    public double X;
    public double Y;
    public IntPtr Axes;
    public uint State;
    private IntPtr _sequence;
    public Gdk.EventSequence? Sequence
    {
        get
        {
            return _sequence == IntPtr.Zero ? null : (Gdk.EventSequence) GLib.Opaque.GetOpaque(_sequence, typeof(Gdk.EventSequence), false);
        }
        set
        {
            _sequence = value == null ? IntPtr.Zero : value.Handle;
        }
    }
    public bool EmulatingPointer;
    private IntPtr _device;
    public Gdk.Device Device
    {
        get
        {
            return GLib.Object.GetObject(_device) as Gdk.Device;
        }
        set
        {
            _device = value == null ? IntPtr.Zero : value.Handle;
        }
    }
    public double XRoot;
    public double YRoot;

    public static EventTouch Zero = new EventTouch();

    public static EventTouch New(IntPtr raw)
    {
        if (raw == IntPtr.Zero)
            return EventTouch.Zero;
        return (EventTouch) Marshal.PtrToStructure(raw, typeof(EventTouch));
    }

    private static GLib.GType GType
    {
        get { return GLib.GType.Pointer; }
    }
}
```

接下来创建一个继承 DrawingArea 的类型，如以下代码的 F 类型

```csharp
class F : DrawingArea
{
   ... // 忽略其他代码
}
```

在 F 的构造函数里面，先使用 AddEvents 函数，告诉 GTK 层，当前的 F 类型所感兴趣的事件。如果没有调用此方法，那将监听事件时不会触发。这里为了简单起见，就添加了足够多的事件

```csharp
class F : DrawingArea
{
    public F()
    {
        AddEvents((int) RequestedEvents);
        ... // 忽略其他代码
    }

    internal const Gdk.EventMask RequestedEvents =
        Gdk.EventMask.EnterNotifyMask
        | Gdk.EventMask.LeaveNotifyMask
        | Gdk.EventMask.ButtonPressMask
        | Gdk.EventMask.ButtonReleaseMask
        | Gdk.EventMask.PointerMotionMask // Move
        | Gdk.EventMask.SmoothScrollMask
        | Gdk.EventMask.TouchMask // Touch
        | Gdk.EventMask.ProximityInMask // Pen
        | Gdk.EventMask.ProximityOutMask // Pen
        | Gdk.EventMask.KeyPressMask
        | Gdk.EventMask.KeyReleaseMask;
}
```

接着在构造函数监听触摸事件，如以下代码

```csharp
class F : DrawingArea
{
    public F()
    {
        AddEvents((int) RequestedEvents);
        TouchEvent += F_TouchEvent;
    }

    private unsafe void F_TouchEvent(object o, TouchEventArgs args)
    {
    }
}
```

从 TouchEventArgs 里的指针可以转换出 EventTouch 结构体，如以下代码

```csharp
class F : DrawingArea
{
    public F()
    {
        AddEvents((int) RequestedEvents);
        TouchEvent += F_TouchEvent;
    }

    private unsafe void F_TouchEvent(object o, TouchEventArgs args)
    {
        var eventTouch = EventTouch.New(args.Event.Handle);
    }
}
```

接下来通过 EventTouch 的 Device 可以获取 Axes 数组的长度。由于 EventTouch 定义的 Axes 是一个 double 类型的指针，如果没有从 Device 获取其长度，那可能出现越界问题

```csharp
            var device = eventTouch.Device;
            var numAxes = device.NumAxes;

            Console.WriteLine($"NumAxes={numAxes} Id={eventTouch.Sequence?.Handle ?? -1}");
```

拿到数组长度，即可将整个数组的内容输出到控制台，如以下代码

```csharp
            var axes = new Span<double>((void*) eventTouch.Axes, numAxes);

            Console.WriteLine("=================");
            for (int i = 0; i < numAxes; i++)
            {
                Console.WriteLine($"[{i}] {axes[i]}");
            }
            Console.WriteLine("=================");
```

根据实际实验，可以看到第 3、4 项就是触摸的尺寸，且是一个被压缩到 0-1 之间的浮点数。可以使用分辨计算 x 和 y 的缩放比从而获取到像素单位的触摸尺寸

```csharp
            if (numAxes >= 5)
            {
                var radioX = eventTouch.XRoot / axes[0];
                var radioY = eventTouch.YRoot / axes[1];

                var rawWidth = axes[3];
                var rawHeight = axes[4];

                var width = rawWidth * radioX;
                var height = rawHeight * radioY;

                Console.WriteLine($"Width={width} Height={height}");
            }
```

通过以上方法即可获取到触摸点的宽度和高度

最后，将 F 类型的对象加入到窗口里面显示，代码如下

```csharp
internal class Program
{
    [STAThread]
    static void Main(string[] args)
    {
        Application.Init();

        _app = new Application("org.Samples.Samples", GLib.ApplicationFlags.None);
        _app.Register(GLib.Cancellable.Current);

        _win = new MainWindow("Demo Window");
        _app.AddWindow(_win);
        _win.ShowAll();
        Application.Run();
    }

    private static Application? _app;
    private static Window? _win;
}

class MainWindow : Window
{
    public MainWindow(string title) : base(WindowType.Toplevel)
    {
        WindowPosition = WindowPosition.Center;
        DefaultSize = new Size(600, 600);

        Child = new F();
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/066cae4e4f6aa4f31d3e43eca9c278aa7b546b60/WarheelaigeQekeyelyai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/066cae4e4f6aa4f31d3e43eca9c278aa7b546b60/WarheelaigeQekeyelyai) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 066cae4e4f6aa4f31d3e43eca9c278aa7b546b60
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 066cae4e4f6aa4f31d3e43eca9c278aa7b546b60
```

获取代码之后，进入 WarheelaigeQekeyelyai 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
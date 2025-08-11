# dotnet X11 获取多屏 edid 信息

故事的背景我在一个双屏设备上，我想要获取每个显示器屏幕对应的 EDID 信息。我在一台麒麟系统的设备上，通过 RandR 的方式获取 EDID 信息，进而读取屏幕物理设备信息

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

我需要获取准确的显示器屏幕关联的设备信息，在屏幕对应的 Edid 信息里面记录了我需要的物理设备信息。我尝试通过 `/sys/class/drm/` 路径读取，但遇到了关联问题，不知道哪个 Edid 文件应该对应哪个屏幕。我不想去猜测，于是询问了 [Handsome08](https://github.com/Handsome08 ) 了解到了使用 XRRGetOutputProperty 获取 Edid 数据的方法。具体的方法如下

通过 XRRGetMonitors 方法获取当前设备的每个显示器屏幕信息，其方法定义代码如下

```csharp
        const string libX11Randr = "libXrandr.so.2";

        [DllImport(libX11Randr)]
        public static extern XRRMonitorInfo* XRRGetMonitors(IntPtr dpy, IntPtr window, bool get_active, out int nmonitors);
```

返回的 XRRMonitorInfo 结构体定义如下

```csharp
    public unsafe struct XRRMonitorInfo
    {
        public IntPtr Name;
        public int Primary;
        public int Automatic;
        public int NOutput;
        public int X;
        public int Y;
        public int Width;
        public int Height;
        public int MWidth;
        public int MHeight;
        public IntPtr* Outputs;
    }
```

为了方便上层调用，我将其再次封装，封装了 MonitorInfo 结构体，代码如下

```csharp
public unsafe struct MonitorInfo
{
    public IntPtr Name;
    public bool IsPrimary;
    public int X;
    public int Y;
    public int Width;
    public int Height;
    public IntPtr[] Outputs;
    public IntPtr Display { get; init; }

    public string? GetNameText()
    {
        var namePtr = XGetAtomName(Display, Name);
        var name = Marshal.PtrToStringAnsi(namePtr);
        XFree(namePtr);
        return name;
    }

    public override string ToString()
    {
        var name = GetNameText();

        return $"{name}({Name}) IsPrimary={IsPrimary} XY={X},{Y} WH={Width},{Height}";
    }
}
```

于是获取屏幕信息的代码就可以这么写

```csharp
// Copy from https://github.com/AvaloniaUI/Avalonia \src\Avalonia.X11\Screens\X11Screen.Providers.cs

public class Randr15ScreensImpl
{
    public Randr15ScreensImpl(nint display, nint rootWindow)
    {
        _display = display;
        var eventWindow = CreateEventWindow(display, rootWindow);
        _window = eventWindow;

        XRRSelectInput(display, _window, RandrEventMask.RRScreenChangeNotify);
    }

    public unsafe MonitorInfo[] GetMonitorInfos()
    {
        XRRMonitorInfo* monitors = XRRGetMonitors(_display, _window, true, out var count);
        var screens = new MonitorInfo[count];
        for (var c = 0; c < count; c++)
        {
            var mon = monitors[c];

            var outputs = new nint[mon.NOutput];

            for (int i = 0; i < outputs.Length; i++)
            {
                outputs[i] = mon.Outputs[i];
            }

            screens[c] = new MonitorInfo()
            {
                Name = mon.Name,
                IsPrimary = mon.Primary != 0,
                X = mon.X,
                Y = mon.Y,
                Width = mon.Width,
                Height = mon.Height,
                Outputs = outputs,
                Display = _display,
            };
        }

        return screens;
    }

    private readonly IntPtr _display;

    private readonly IntPtr _window;
}
```

以上代码是从 Avalonia 项目拷贝的。经过了 [SeWZC](https://github.com/SeWZC) 的考证，传入 XRRGetMonitors 的窗口应该是 RootWindow 窗口，然而在本文这里和 Avalonia 这里都传入的是一个 EventWindow 窗口，且传入 EventWindow 窗口能拿到正确的值，十分有趣，更底层原因我就没有继续调查了

以上代码的 CreateEventWindow 方法的实现如下

```csharp
        public static IntPtr CreateEventWindow(nint display, nint rootWindow)
        {
            var win = XCreateSimpleWindow(display, rootWindow,
                0, 0, 1, 1, 0, IntPtr.Zero, IntPtr.Zero);
            return win;
        }

        [DllImport(libX11)]
        public static extern IntPtr XCreateSimpleWindow(IntPtr display, IntPtr parent, int x, int y, int width,
            int height, int border_width, IntPtr border, IntPtr background);

        const string libX11 = "libX11.so.6";
```

拿到 MonitorInfo 对象之后，可以看到里面有很多个属性，其中的 Outputs 属性就是本文的重点

在这里我编写一个循环将其逐个取出，其中可能有一个就是包含了 EDID 信息，代码如下

```csharp
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);
var rootWindow = XDefaultRootWindow(display);

var randr15ScreensImpl = new Randr15ScreensImpl(display, rootWindow);
MonitorInfo[] monitorInfos = randr15ScreensImpl.GetMonitorInfos();

for (var i = 0; i < monitorInfos.Length; i++)
{
    MonitorInfo monitorInfo = monitorInfos[i];
    Console.WriteLine(monitorInfo);

    OutputEdidInfo(monitorInfo);
}
```

在 OutputEdidInfo 方法里面，咱将进行 EDID 解析逻辑

通过 XRRListOutputProperties 方法读取 Outputs 里面的每一项，如果某一项中读取到的属性包含了 EDID Atom 内容，则证明当前项就是 EDID 信息

其代码如下

```csharp
unsafe void OutputEdidInfo(MonitorInfo monitorInfo)
{
    var edidAtom = XInternAtom(display, "EDID", only_if_exists: true);
    var anyPropertyTypeAtom = XInternAtom(display, "AnyPropertyType", only_if_exists: true);
    const nint XA_INTEGER = 19;

    for (var i = 0; i < monitorInfo.Outputs.Length; i++)
    {
        var rrOutput = monitorInfo.Outputs[i];
        if (rrOutput == IntPtr.Zero)
        {
            continue;
        }

        var properties = XRRListOutputProperties(display, rrOutput, out var propertyCount);

        IntPtr prop = 0;

        try
        {
            var hasEDID = false;
            for (var pc = 0; pc < propertyCount; pc++)
            {
                if (properties[pc] == edidAtom)
                {
                    hasEDID = true;
                    break;
                }
            }

            if (!hasEDID)
            {
                Console.WriteLine($"Output {rrOutput} does not have EDID property.");
                continue;
            }

            ... // 忽略其他代码
        }
        finally
        {
            XLib.XFree(prop);
            XLib.XFree(new IntPtr(properties));
        }
    }
}
```

如果当前项的 hasEDID 为 true 则通过 XRRGetOutputProperty 读取属性的值，代码如下

```csharp
            // Length of a EDID-Block-Length(128 bytes), XRRGetOutputProperty multiplies offset and length by 4
            const int EDIDStructureLength = 32;

            XRRGetOutputProperty(display, rrOutput, edidAtom, 0, EDIDStructureLength, false, false,
                anyPropertyTypeAtom, out IntPtr actualType, out int actualFormat, out int nItems, out long bytesAfter,
                out prop);

            // https://gitlab.gnome.org/GNOME/mutter/-/blame/3.29.90/src/backends/x11/meta-output-xrandr.c

            if (actualType != XA_INTEGER)
            {
                continue;
            }

            if (actualFormat != 8) // Expecting a byte array
            {
                continue;
            }

            Span<byte> edid = new Span<byte>((void*) prop, (int) bytesAfter);
```

此时即可读取到 edid 二进制信息。此时的 edid 二进制信息还需要进一步的解析才能获取内容。如何解析 edid 就不在本文范围内了，大家可以使用自己喜欢的方式进行解析。本文这里只是做了简单的内容解析，解析出了屏幕的物理尺寸信息。解析代码封装在 EdidInfo 结构体里面，如果大家感兴趣，可以到本文末尾找到本文所有代码的下载方法，下载代码了解 Edid 解析逻辑

解析之后将输出物理设备信息，代码如下

```csharp
            Span<byte> edid = new Span<byte>((void*)prop, (int)bytesAfter);

            ReadEdidInfoResult edidInfoResult = EdidInfo.ReadEdid(edid);
            if (edidInfoResult.IsSuccess)
            {
                EdidInfo edidInfo = edidInfoResult.EdidInfo;

                Console.WriteLine($"EDID Info: ManufacturerName={edidInfo.ManufacturerName} MonitorPhysical={edidInfo.BasicDisplayParameters.MonitorPhysicalWidth.Value}x{edidInfo.BasicDisplayParameters.MonitorPhysicalHeight.Value}cm");
            }
            else
            {
                Console.WriteLine($"解析 Edid 失败 {edidInfoResult.ErrorMessage}");
            }
```

在我的双屏设备上运行，可以看到大概如下输出信息。我的 `DisplayPort-1` 是主屏，放在右边，是一个 165x93cm 的 75 寸大屏幕。副屏是 `DisplayPort-0` 放在左边，是一个 190x107cm 的更大的屏幕

```
DisplayPort-1(343) IsPrimary=True XY=1920,309 WH=1920,1080
EDID Info: ManufacturerName=IWB MonitorPhysical=165x93cm
DisplayPort-0(344) IsPrimary=False XY=0,0 WH=1920,1080
EDID Info: ManufacturerName=IWB MonitorPhysical=190x107cm
```

通过以上输出，对比我的物理设备，发现可以对应上，通过此方法比从 `/sys/class/drm/` 路径下读取更好，至少不用去猜路径名。很多设备上，都可以在 `/sys/class/drm/` 文件夹内找到和 XRRGetMonitors 返回的显示器名对应的设备，但这取决于驱动，不一定能对应上。能从 XRRGetOutputProperty 获取到的 Edid 信息才能完全对应

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/46db729ca3aaa4d73169d07e903c96f0aa2f7fee/X11/FelocerebeWirolerco) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/46db729ca3aaa4d73169d07e903c96f0aa2f7fee/X11/FelocerebeWirolerco) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 46db729ca3aaa4d73169d07e903c96f0aa2f7fee
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 46db729ca3aaa4d73169d07e903c96f0aa2f7fee
```

获取代码之后，进入 X11/FelocerebeWirolerco 文件夹，即可获取到源代码

更多 X11 技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
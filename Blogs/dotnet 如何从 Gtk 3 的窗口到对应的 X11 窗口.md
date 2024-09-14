本文将告诉大家如何在 Gtk3 的 Gtk.Window 或 Gdk.Window 里面获取到对应的 X11 窗口 XID 号

<!--more-->


<!-- CreateTime:2024/05/15 07:25:32 -->

<!-- 发布 -->
<!-- 博客 -->

记录本文是因为我在这里踩了很多坑，核心问题就是 GTK 有很多个版本，我开始找的全是使用 GTK 2 的 `gdk_x11_drawable_get_xid` 方法，而不是 GtkSharp 3.24 对应的 GTK 3 的方法

以上的 [gdk_x11_drawable_get_xid](https://www.manpagez.com/html/gdk2/gdk2-2.24.29/gdk2-X-Window-System-Interaction.php#gdk-x11-drawable-get-xid) 方法需要构建传入 GdkDrawable 指针，让我弄错为使用 `gtk_widget_get_window` 方法去获取其 gdk 窗口，于是错误就更加诡异

通过阅读文档发现了以下的 gtk 架构图，即 gtk 的窗口和 gdk 窗口是不相同的，可以通过 `gtk_widget_get_window` 方法获取，在 C# dotnet 里面可直接使用 Gtk.Window 的 Window 属性，更多请参阅：<https://en.wikipedia.org/wiki/GDK>

<!-- ![](image/dotnet 如何从 Gtk 3 的窗口到对应的 X11 窗口/dotnet 如何从 Gtk 3 的窗口到对应的 X11 窗口0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240915070909622-626182065.png)

从 Gtk 的 Window 窗口获取 Gdk 的 Window 窗口，可使用以下简单代码获取

```csharp
        Gtk.Window window = xxx;
        Gdk.Window gdkWindow = window.Window;
```

获取 Gdk 窗口指针，可通过 Handle 属性获取，如以下代码

```csharp
        Gdk.Window gdkWindow = window.Window;

        var handle = gdkWindow.Handle;
```

以上获取的 handle 指针与 `var windowHandle = gtk_widget_get_window(gtkWindow.Handle);` 获取的 `windowHandle` 是相同的，因为在 GtkSharp 的 Widget.cs 就是如此实现的

```csharp
namespace Gtk;

public partial class Widget 
{
    ... // 忽略其他代码

		[GLib.Property ("window")]
		public Gdk.Window Window 
		{
			get  
			{
				IntPtr raw_ret = gtk_widget_get_window(Handle);
				Gdk.Window ret = GLib.Object.GetObject(raw_ret) as Gdk.Window;
				return ret;
			}
			set  
			{
				gtk_widget_set_window(Handle, value == null ? IntPtr.Zero : value.Handle);
			}
		}

		[UnmanagedFunctionPointer (CallingConvention.Cdecl)]
		delegate IntPtr d_gtk_widget_get_window(IntPtr raw);
		static d_gtk_widget_get_window gtk_widget_get_window = FuncLoader.LoadFunction<d_gtk_widget_get_window>(FuncLoader.GetProcAddress(GLibrary.Load(Library.Gtk), "gtk_widget_get_window"));
}

public partial class Container : Gtk.Widget
{
   ... // 忽略其他代码
}

public partial class Bin : Gtk.Container
{
   ... // 忽略其他代码
}

public partial class Window : Gtk.Bin
{
   ... // 忽略其他代码
}
```

使用 [gdk_x11_window_get_xid](https://docs.gtk.org/gdk3-x11/method.X11Window.get_xid.html) 方法即可正确的从 gdk 窗口获取到对应的 X11 窗口的 XID 值

为了方便使用 gdk_x11_window_get_xid 方法，以下照 GtkSharp 进行一些代码定义

```csharp
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    delegate IntPtr d_gdk_x11_window_get_xid(IntPtr gdkWindow);

    private static d_gdk_x11_window_get_xid gdk_x11_window_get_xid =
        LoadFunction<d_gdk_x11_window_get_xid>("libgdk-3.so.0", "gdk_x11_window_get_xid");

    private static T LoadFunction<T>(string libName, string functionName)
    {
        if (!OperatingSystem.IsLinux())
        {
            // 防止炸调试
            return default(T)!;
        }

        // LoadLibrary
        var libPtr = Linux.dlopen(libName, RTLD_GLOBAL | RTLD_LAZY);

        Console.WriteLine($"Load {libName} ptr={libPtr}");

        // GetProcAddress
        var procAddress = Linux.dlsym(libPtr, functionName);
        Console.WriteLine($"Load {functionName} ptr={procAddress}");
        return Marshal.GetDelegateForFunctionPointer<T>(procAddress);
    }

    private const int RTLD_LAZY = 0x0001;
    private const int RTLD_GLOBAL = 0x0100;

    private class Linux
    {
        [DllImport("libdl.so.2")]
        public static extern IntPtr dlopen(string path, int flags);

        [DllImport("libdl.so.2")]
        public static extern IntPtr dlsym(IntPtr handle, string symbol);
    }
```

接着在窗口的 Show 方法之后，即可获取到对应的 X11 窗口

```csharp
    protected override void OnShown()
    {
        base.OnShown(); // 在这句话调用之前 window.Window 是空

        Window window = this;
        Gdk.Window gdkWindow = window.Window;

        if (gdkWindow is null)
        {
            // 确保 base.OnShown 调用
            Console.WriteLine($"gdkWindow is null");
            return;
        }

        var x11 = gdk_x11_window_get_xid(gdkWindow.Handle);
        Console.WriteLine($"X11 窗口 0x{x11:x2}");
    }
```

通过以上代码输出的 X11 窗口的 XID 号，可以同步在命令行输入进 xwininfo 命令里面。比如我这里输出的是 `X11 窗口 0x5600003` 的值

打开另一个命令行，输入以下命令，将 XID 传入 xwininfo 命令，即可看到显示的窗口标题和当前运行的窗口是相同的

<!-- ![](image/dotnet 如何从 Gtk 3 的窗口到对应的 X11 窗口/dotnet 如何从 Gtk 3 的窗口到对应的 X11 窗口1.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240915070910010-233821819.png)

我核心踩坑就是搜到的是 GTK 2 的使用方法，以及将 gtk 的窗口当成 gdk 的窗口传入方法

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/4ae5a45eb65cab5a3b9f8991852be9602dee6533/LejarkeebemCowakiwhanar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/4ae5a45eb65cab5a3b9f8991852be9602dee6533/LejarkeebemCowakiwhanar) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4ae5a45eb65cab5a3b9f8991852be9602dee6533
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 4ae5a45eb65cab5a3b9f8991852be9602dee6533
```

获取代码之后，进入 LejarkeebemCowakiwhanar 文件夹，即可获取到源代码


更多 GTK 开发请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

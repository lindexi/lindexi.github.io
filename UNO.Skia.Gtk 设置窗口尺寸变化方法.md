# UNO.Skia.Gtk 设置窗口尺寸变化方法

本文记录一个简单的在 UNO.Skia.Gtk 应用里面，配置 GTK 平台修改窗口尺寸的方法

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

为了全平台通用性，推荐是走定义接口加平台注入的方式。定义的接口如下

```csharp
public interface IWindowActivator
{
    void ResizeMainWindow(Size size);
}
```

这里为了方便起见，直接使用静态属性注入方法，如以下定义

```csharp
public static class WindowHelper
{
    public static IWindowActivator WindowActivator { get; set; } = null!;
}
```

在 GTK 平台上定义具体的修改窗口大小的实现

```csharp
    class WindowActivator : IWindowActivator
    {
        public GtkHost GtkHost { get; set; } = null!;

        public void ResizeMainWindow(Size size)
        {
            var nativeWindow = GtkHost.Window;

            nativeWindow.Resize((int) size.Width, (int) size.Height);
        }
    }
```

完成定义之后，在 GTK 应用初始化进行注入属性，如下面代码

```csharp
 public static void Main(string[] args)
 {
     ExceptionManager.UnhandledException += delegate (UnhandledExceptionArgs expArgs)
     {
         Console.WriteLine("GLIB UNHANDLED EXCEPTION" + expArgs.ExceptionObject.ToString());
         expArgs.ExitApplication = true;
     };

     var windowActivator = new WindowActivator();
     WindowHelper.WindowActivator = windowActivator;

     var host = new GtkHost(() =>
     {
         var app = new AppHead();
         return app;
     });
     windowActivator.GtkHost = host;
     host.Run();
 }
```

接下来即可在通用的全平台代码里面利用 WindowHelper 辅助修改 GTK 的窗口

```csharp
    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        var size = new Size(Random.Shared.Next(200, 1000), Random.Shared.Next(200, 1000));
        WindowHelper.WindowActivator.ResizeMainWindow(size);
    }
```

更进一步的优化是在各个平台里面都实现修改窗口尺寸的具体实现，如此即可让全平台部分的代码可以实现使用相同的代码，用接口加多态实现完成对各个平台的设置窗口

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/f91082b406f3ac1e1a89c536263fd62c7d99be47/LacebayjeejiBehebilawla) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f91082b406f3ac1e1a89c536263fd62c7d99be47/LacebayjeejiBehebilawla) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f91082b406f3ac1e1a89c536263fd62c7d99be47
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f91082b406f3ac1e1a89c536263fd62c7d99be47
```

获取代码之后，进入 LacebayjeejiBehebilawla 文件夹
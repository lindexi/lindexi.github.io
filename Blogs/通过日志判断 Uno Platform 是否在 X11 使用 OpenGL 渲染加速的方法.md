---
title: 通过日志判断 Uno Platform 是否在 X11 使用 OpenGL 渲染加速的方法
description: 本文告诉大家如何在 UNO 里面，如何通过日志信息判断是否在 Linux 的 X11 平台上使用 OpenGL 渲染加速
tags: UNO,X11
category: 
---

<!-- CreateTime:2024/07/24 07:19:59 -->

<!-- 发布 -->
<!-- 博客 -->

本文的方法适用于 UNO 的 5.2.175 版本，其他版本还请大家自行测试

需要先开启 UNO 的日志输出，即默认的 App.xaml.cs 里的 InitializeLogging 方法需要确保被调用且里面的代码需要被执行。更具体的测试就是默认选择使用 Debug 模式，或者去掉条件编译符

再添加日志过滤，如以下代码

```csharp
            builder.AddFilter("Uno.WinUI.Runtime.Skia.X11.X11OpenGLRenderer", LogLevel.Trace);
```

修改之后的 InitializeLogging 方法的代码大概如下

```csharp
    public static void InitializeLogging()
    {
        // Logging is disabled by default for release builds, as it incurs a significant
        // initialization cost from Microsoft.Extensions.Logging setup. If startup performance
        // is a concern for your application, keep this disabled. If you're running on the web or
        // desktop targets, you can use URL or command line parameters to enable it.
        //
        // For more performance documentation: https://platform.uno/docs/articles/Uno-UI-Performance.html

        var factory = LoggerFactory.Create(builder =>
        {
#if __WASM__
            builder.AddProvider(new global::Uno.Extensions.Logging.WebAssembly.WebAssemblyConsoleLoggerProvider());
#elif __IOS__ || __MACCATALYST__
            builder.AddProvider(new global::Uno.Extensions.Logging.OSLogLoggerProvider());
#else
            builder.AddConsole();
#endif

            // Exclude logs below this level
            builder.SetMinimumLevel(LogLevel.Information);

            // Default filters for Uno Platform namespaces
            builder.AddFilter("Uno", LogLevel.Warning);
            builder.AddFilter("Windows", LogLevel.Warning);
            builder.AddFilter("Microsoft", LogLevel.Warning);

            builder.AddFilter("Uno.WinUI.Runtime.Skia.X11.X11OpenGLRenderer", LogLevel.Trace);

            // Generic Xaml events
            // builder.AddFilter("Microsoft.UI.Xaml", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.VisualStateGroup", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.StateTriggerBase", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.UIElement", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.FrameworkElement", LogLevel.Trace );

            // Layouter specific messages
            // builder.AddFilter("Microsoft.UI.Xaml.Controls", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.Controls.Layouter", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.Controls.Panel", LogLevel.Debug );

            // builder.AddFilter("Windows.Storage", LogLevel.Debug );

            // Binding related messages
            // builder.AddFilter("Microsoft.UI.Xaml.Data", LogLevel.Debug );
            // builder.AddFilter("Microsoft.UI.Xaml.Data", LogLevel.Debug );

            // Binder memory references tracking
            // builder.AddFilter("Uno.UI.DataBinding.BinderReferenceHolder", LogLevel.Debug );

            // DevServer and HotReload related
            // builder.AddFilter("Uno.UI.RemoteControl", LogLevel.Information);

            // Debug JS interop
            // builder.AddFilter("Uno.Foundation.WebAssemblyRuntime", LogLevel.Debug );
        });

        global::Uno.Extensions.LogExtensionPoint.AmbientLoggerFactory = factory;

#if HAS_UNO
        global::Uno.UI.Adapter.Microsoft.Extensions.Logging.LoggingAdapter.Initialize();
#endif
    }
```

可在 Program.cs 里添加测试代码，如下面代码，如果有命令行参数，则不开启 OpenGL 渲染加速，代码如下

```csharp
public class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        App.InitializeLogging();

        if (args.Length > 0)
        {
            FeatureConfiguration.Rendering.UseOpenGLOnX11 = false;
        }

        var host = SkiaHostBuilder.Create()
            .App(() => new App())
            .UseX11()
            .UseLinuxFrameBuffer()
            .UseMacOS()
            .UseWindows()
            .Build();

        host.Run();
    }
}
```

尝试运行以上代码，在 Linux 上进行运行。在开启 OpenGL 渲染加速时，可在控制台看到如下输出代码

```
trce: Uno.WinUI.Runtime.Skia.X11.X11OpenGLRenderer[0]
      Render 0
trce: Uno.WinUI.Runtime.Skia.X11.X11OpenGLRenderer[0]
      Render 1
trce: Uno.WinUI.Runtime.Skia.X11.X11OpenGLRenderer[0]
      Render 2
trce: Uno.WinUI.Runtime.Skia.X11.X11OpenGLRenderer[0]
      Render 3
```

通过阅读 UNO 的源代码，可以了解到上述的日志输出对应的代码如下

```csharp
namespace Uno.WinUI.Runtime.Skia.X11
{
    internal class X11OpenGLRenderer : IX11Renderer, IDisposable
    {
        ... // 忽略其他代码

        void IX11Renderer.InvalidateRender()
        {
            ... // 忽略其他代码

            if (this.Log().IsEnabled(LogLevel.Trace))
            {
                this.Log().Trace($"Render {_renderCount++}");
            }
            ... // 忽略其他代码
        }
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7de15e7b026989eab007b3ca4b38f56f334f175b/UnoDemo/BallnallqeebairCejaiwakeneadi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7de15e7b026989eab007b3ca4b38f56f334f175b/UnoDemo/BallnallqeebairCejaiwakeneadi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7de15e7b026989eab007b3ca4b38f56f334f175b
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7de15e7b026989eab007b3ca4b38f56f334f175b
```

获取代码之后，进入 UnoDemo/BallnallqeebairCejaiwakeneadi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )


# dotnet 解决 Avalonia 在 OpenKylin 麒麟系统运行找不到默认字体启动失败

本文记录 Avalonia 应用在 OpenKylin 麒麟系统运行找不到默认字体启动失败的解决方法

<!--more-->


<!-- CreateTime:2024/04/13 07:05:19 -->

<!-- 发布 -->
<!-- 博客 -->

本文的解决方法由 [walterlv](https://blog.walterlv.com) 提供，我只是代为记录的工具人

在 OpenKylin 系统启动 Avalonia 应用失败，异常信息是 System.InvalidOperationException: Default font family name can't be null or empty.

启动失败的异常堆栈信息如下

```
System.InvalidOperationException: Default font family name can't be null or empty.
  at Avalonia.Media.FontManager..ctor(IFontManagerImpl platformImpl)
  at Avalonia.Media.FontManager.get_Current()
  ...
```

修复方法，添加默认字体配置，代码如下

```csharp
        appBuilder.With(new FontManagerOptions()
        {
            DefaultFamilyName = "Noto Sans CJK SC",
            FontFallbacks =
            [
                new FontFallback { FontFamily = "文泉驿正黑" },
                new FontFallback { FontFamily = "DejaVu Sans" },
            ],
        });
```

在我代码里面，加上之后的 Program.cs 代码如下。以下代码使用的是 11.0.6 的 Avalonia 版本

```csharp
internal class Program
{
    // Initialization code. Don't use any Avalonia, third-party APIs or any
    // SynchronizationContext-reliant code before AppMain is called: things aren't initialized
    // yet and stuff might break.
    [STAThread]
    public static void Main(string[] args) =>
        BuildAvaloniaApp()
            .StartWithClassicDesktopLifetime(args);

    // Avalonia configuration, don't remove; also used by visual designer.
    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>()
            .UsePlatformDetect()
            .WithInterFont()
            .LogToTrace()
            // 修复麒麟丢失字体
            .With(new FontManagerOptions()
            {
                DefaultFamilyName = "Noto Sans CJK SC",
                FontFallbacks =
                [
                    new FontFallback { FontFamily = "文泉驿正黑" },
                    new FontFallback { FontFamily = "DejaVu Sans" },
                ],
            });
}
```

参考文档：

- [OpenKylin上运行Avalonia应用 - louzi - 博客园](https://www.cnblogs.com/louzixl/p/17631717.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
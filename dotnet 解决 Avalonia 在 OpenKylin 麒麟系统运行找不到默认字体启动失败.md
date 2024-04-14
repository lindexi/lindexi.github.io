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
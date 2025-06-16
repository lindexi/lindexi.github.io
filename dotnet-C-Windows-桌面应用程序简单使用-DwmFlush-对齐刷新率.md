
# dotnet C# Windows 桌面应用程序简单使用 DwmFlush 对齐刷新率

在 Windows 桌面应用程序里面，可以简单地使用 DwmFlush 方法来与 DWM 对齐刷新率

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

桌面管理器 DWM （DWM Desktop Window Manager）是一个古老的机制，在 Windows 上的 DWM 将会收集每个窗口的绘制内容，然后将其进行合成，最后输出到屏幕上。通过 DWM 机制，可以避免应用程序直接将画面输出到屏幕上，允许多个应用程序之间叠加出绚丽的窗口化效果

调用 Dwmapi.dll 提供的 DwmFlush 方法，可以让调用方阻塞卡住，等待当前次渲染刷新完成之后才继续后续逻辑。通过这样的机制，就可以实现和 DwmFlush 对齐刷新率，其测试代码如下

```csharp
using System.ComponentModel;
using System.Diagnostics;
using System.Runtime.InteropServices;

var stopwatch = new Stopwatch();

while (true)
{
    stopwatch.Restart();
    var hResult = DwmFlush();
    stopwatch.Stop();

    Console.WriteLine($"Elapsed={stopwatch.ElapsedMilliseconds}ms HResult={hResult}");

    if (hResult != 0)
    {
        // Fail
        break;
    }
}

Console.WriteLine("Hello, World!");

[DllImport("Dwmapi.dll")]
static extern int DwmFlush();
```

预期运行之后，将可以在控制台看到比较恒定的频率输出。在我的设备上是 60Hz 的刷新率，输出大概就是 16 毫秒。或偶尔跳几次 15 或 17 毫秒。但如果此时在循环里面多做一些事情，则可以看到其输出的毫秒数变短，但其控制台输出频率不变

由于 CPU/系统 的线程调度影响，平均的暂停时间预期会比单个 DwmFlush 略高。即在从 DwmFlush 方法释放出来之后，可能需要过一段时间才能获取 CPU 执行权，此时会导致时间略长

此方法也是 WPF 渲染刷新对齐的核心机制之一，如 WPF 框架的源代码所示：<https://github.com/dotnet/wpf/blob/b6316f1cfeb486229ca61368df3bc203da482948/src/Microsoft.DotNet.Wpf/src/WpfGfx/core/uce/rendertargetmanager.cpp#L1171>

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9720d03f2dad4cc84682b784dd56c2b9fed8bb3f/Workbench/DehemrallneFurwaiwakifije) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/9720d03f2dad4cc84682b784dd56c2b9fed8bb3f/Workbench/DehemrallneFurwaiwakifije) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9720d03f2dad4cc84682b784dd56c2b9fed8bb3f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9720d03f2dad4cc84682b784dd56c2b9fed8bb3f
```

获取代码之后，进入 Workbench/DehemrallneFurwaiwakifije 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
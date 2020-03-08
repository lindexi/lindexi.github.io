# dotnet core 输出调试信息到 DebugView 软件

本文告诉大家如何在 dotnet core 输出调试信息到 DebugView 软件

<!--more-->
<!-- CreateTime:2019/11/29 10:14:03 -->


在之前告诉小伙伴，如何在 WPF 输出调试信息到 DebugView 软件，请看文章 [WPF 调试 获得追踪输出](https://blog.lindexi.com/post/WPF-%E8%B0%83%E8%AF%95-%E8%8E%B7%E5%BE%97%E8%BF%BD%E8%B8%AA%E8%BE%93%E5%87%BA.html )

那么如何在 dotnet core 也输入信息到 DebugView 软件？

实际上和在 WPF 的做法一样，通过 Trace 输出就可以

首先引用命名空间

```csharp
using System.Diagnostics;

```

然后使用下面的代码就可以输出到 DebugView 软件

```csharp
            Trace.WriteLine("欢迎访问我博客 https://blog.lindexi.com 里面有大量 UWP WPF 博客");

```

如果此时在 VisualStudio 进行调试，同时打开 [DebugView](https://docs.microsoft.com/en-us/sysinternals/downloads/debugview ) 会发现 DebugView 没有任何输出

但是如果是不进行 VisualStudio 附加，通过命令行启动 dotnet core 程序，那么就可以在 DebugView 看到输出，因为在附加 VisualStudio 的时候会输出到 VS 所以就看不到

建议有一些输出通过 Trace 输出，这样如果有用户说在他的设备无法运行，可以通过 DebugView 放在用户的设备，这样可以看到输出

另外如果需要使用 Trace 输出要求勾选生成的 Trace 命令

![](http://image.acmx.xyz/lindexi%2F201851615734398.jpg)

官网：[DebugView](https://docs.microsoft.com/en-us/sysinternals/downloads/debugview )

[调试日志大师(DebugView的增强版)-CSDN下载](https://download.csdn.net/download/wg_duan/943900 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

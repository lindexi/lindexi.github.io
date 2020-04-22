# WinForms 下的高性能笔迹方法

在 WPF 中可以通过 StylusPlugIn 的方式快速从触摸线程拿到触摸数据，而 WinForms 没有这个机制，但是可以通过 Microsoft.Ink 组件和 WPF 相同在 RealTimeStylus 服务拿到触摸点

<!--more-->
<!-- CreateTime:4/20/2020 5:00:00 PM -->

<!-- csdn -->

本文的例子放在微软的官方例子里面，请看 [GitHub](https://github.com/microsoft/Windows-classic-samples/tree/8f31b1ff79d669b4ba9609f2640635b3b8a9e0a4/Samples/Win7Samples/Touch/MTScratchpadRTStylus/CS ) 代码

本文不会告诉大家代码如何写，因为看微软的代码就知道了，本文主要告诉大家为什么这样做的触摸收集足够快

在 Windows 里面，会通过 `WM_TOUCH` 的消息发送触摸的信息，但是通过消息的方式不够快。当然这句话不是说消息不快，而是和 Wisptis 服务对比还是不够快


从微软的笔和触摸服务里面获取的方法是需要使用复杂的方法才能获取到内容，大概的获取方法就是从共享内存里面读取值，此时的读取速度将比消息快

在 csproj 里面引用 Microsoft.Ink 组件，将可以获取 RealTimeStylus 的封装方法，不需要写一个 PenImc 的组件。但是通过这个方法将拿到十分底层的内容，需要小伙伴自己处理

```csharp
    <Reference Include="Microsoft.ink, Version=6.1.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35">
      <SpecificVersion>True</SpecificVersion>
    </Reference>
```

我推荐下载微软的代码，然后构建，尝试运行，然后对比一下性能。这个方法能拿到最快的触摸信息，但是这个方法优化的仅是触摸收集，而对渲染没有做多少优化

更多请看微软文档 [Windows Touch Scratchpad using the Real-Time Stylus Sample (C#) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/wintouch/windows-touch-scratchpad-using-the-real-time-stylus-in-c--sample--mtscratchpadrtstyluscs- )

在 WPF 中默认的 StylusPlugIn 就使用了这部分优化，不需要额外的引用组件，如何在 WPF 中做高性能笔请看 [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

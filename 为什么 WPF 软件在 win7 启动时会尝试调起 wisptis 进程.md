# 为什么 WPF 软件在 win7 启动时会尝试调起 wisptis 进程

我看到一个问题是在 win7 系统上，如果开机启动的软件是 WPF 软件，而这个 WPF 软件在系统的 wisptis 进程启动之前就启动了，那么 WPF 将会调起 wisptis 进程。而在 wisptis 进程已经启动完成，此时启动 WPF 进程不会再打开新的 wisptis 进程。但是被 WPF 启动的 wisptis 进程存在这样的问题，在触摸屏上 win7 的双指打开右键菜单等功能不可用

<!--more-->
<!-- CreateTime:2020/1/20 16:28:32 -->

<!-- 发布 -->

在 WPF 启动时，将会在 Window 类的 Visibility 修改时调用到 WispLogic.RegisterHwndForInput 方法进行初始化触摸，这部分详细请看 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html) 而在初始化触摸时，需要用到 PenIMC 的逻辑

在 win7 系统上，触摸需要通过 wisptis 进程的辅助才能让 WPF 进程能够完成实时触摸，这里的 wisptis 是 Windows Ink Services Platform Tablet Input Subsystem 进程，用于处理触摸书写等功能。也是 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 的提供，通过一些不靠谱的文档和经验，其实 PenIMC 的核心逻辑就是 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 提供的。上面这句话对或不对我不敢说，只能说用 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 可以实现 PenIMC 的效果，而且 API 和参数差不多

那么 PenIMC 又是什么呢？其实 PenIMC 是 `penimc2_v0400.dll` 文件，在不同的版本的 .NET Framework 和系统上这个文件是不同的，包括文件名也不同，看这个文件命名就知道。没错，你可以在 `penimc2_v0400.dll` 文件所在的文件夹找到一堆 penimc 文件。这个文件就是提供给 WPF 的触摸核心 `PenThreadWorker` 的 COM 组件（其实没有文档说这货是纯 COM 组件） 也就是和触摸相关的

也就是在 WPF 窗口打开显示将会初始化触摸，初始化触摸需要依赖 PenIMC 而假设这个库本质是 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 的封装，那么依赖于 win7 的 Windows Ink Services Platform Tablet Input Subsystem 服务，此时在我看不到的代码判断了 wisptis 进程的启动

而为什么 WPF 启动的 wisptis 进程有很多坑？只是启动进程权限问题，更详细我也不知道

规避方法是什么？其实不让触摸执行也就是可以了，但是我如何让 WPF 还能交互？没关系，假装自己是一个古老的应用，只支持鼠标消息就可以啦。但是我想要做多指触摸怎么办？先不要触摸，等待 wisptis 进程启动之后，通过 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html) 方案重新注册一遍触摸

我一开始启动太快了，没关系，我一开始启动的是一个 win32 的启动图，等待后台逻辑判断 wisptis 启动之后，我才打开 WPF 的窗口。根据上面的说法，其实窗口没有修改 Visiliblity 之前是不会初始化触摸的，也就是不会启动 wisptis 进程的

现在 win7 已经不受微软支持了，是时候升级 win10 啦

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

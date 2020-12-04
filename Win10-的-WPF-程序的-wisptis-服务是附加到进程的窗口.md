
# Win10 的 WPF 程序的 wisptis 服务是附加到进程的窗口

在 Win10 下，没有 WISPTIS 服务进程，和 win7 不相同。但是 WPF 依然通过 PENIMC 从 COM 获取实时触摸消息，那么 WPF 是从哪里获取

<!--more-->


<!-- CreateTime:2020/7/30 17:19:49 -->



通过 [WindowDebugger](https://github.com/kkwpsv/WindowDebugger ) 调试工具可以了解在 win10 将会给每个 WPF 进程添加 WISPTIS 窗口，窗口的 Class Name 是 TabletPenServiceHelperClass 请看下图

<!-- ![](image/Win10 的 WPF 程序的 wisptis 服务是附加到进程的窗口/Win10 的 WPF 程序的 wisptis 服务是附加到进程的窗口0.png) -->

![](http://image.acmx.xyz/lindexi%2F20207301722262659.jpg)

更多关于 WPF 触摸请看

[WPF 触摸底层 PenImc 是如何工作的](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%BA%95%E5%B1%82-PenImc-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84.html)

[WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# 浅谈 Windows 桌面端触摸架构演进

我在和小伙伴水触摸相关的坑，说到了上古的触摸，很难和小伙伴统一知识，于是就写了本文用于告诉大家，桌面端的触摸架构是如何一步步演进的

<!--more-->
<!-- CreateTime:2019/11/29 10:20:37 -->

<!-- csdn -->

所有触摸架构都建立在系统之上，和系统版本相关。所以可以通过系统划分。虽然说是触摸架构，但是我能知道的也就是应用层面的接口和编程方法，如果是小伙伴被标题吸引过来的，想看触摸架构，那么请左转官方的 [文档](https://docs.microsoft.com/en-us/windows/win32/wintouch/architectural-overview )

在 XP 之前的系统，在开发行业，触摸屏只有少数游戏才能使用，此时触摸屏不是标准设备，各个应用需要通过端口访问硬件设备。也就是软件没有通过系统，直接和硬件通讯做到触摸。因为没有约定好触摸屏标准，可能有些触摸屏用的是PIN输入，有些用蓝牙输入，在上古的开发者都是非常厉害的，所以实现起来十分诡异。在这里是无法说明架构的，每个公司都可以使用自己的方式开发。请看 [Windows for Pen Computing](https://en.wikipedia.org/wiki/Windows_for_Pen_Computing )

在 XP 的时候，此时触摸屏成为 HID 设备，也就是有了标准。可以在系统上支持触摸屏，我记得在缺少补丁和驱动将会只支持单点触摸，如果需要支持多点触摸，需要额外的补丁或驱动。这部分我没有去查文档，如有错误，请告诉我。在 XP 的触摸存在很多坑，如果要在 XP 上开发支持多点触摸的应用，需要用一些有趣的技术，如[TUIO](http://www.tuio.org/ )等。此时的触摸屏还是作为 Mouse 用，此时我将这一代的触摸架构称为上古的触摸

在 Vista 和 Win7 才算支持多点触摸，此时的应用默认可以收到了 Gestures 消息，如果注册了 [RegisterTouchWindow](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registertouchwindow ) 的窗口将会接收到 [Touch](https://docs.microsoft.com/en-us/windows/win32/wintouch/getting-started-with-multi-touch-messages ) 消息，此时通过 Touch 消息就能做到多指触摸，请看 [Detecting and Tracking Multiple Touch Points](https://docs.microsoft.com/en-us/windows/win32/wintouch/detecting-and-tracking-multiple-touch-points )

在 Win7 的触摸是通过硬件设备作为 HID 设备，通过 HAL 层收集到信息，通过系统驱动进行转发消息。此时的硬件将不是直接连接到软件，中间会经过很多层，此时系统将会处理很多触摸的细节，开发的难度降低。但是这也存在一个坑就是系统的稳定性，如果系统没有正确处理触摸消息和触摸消息的转发，那么将会让应用或系统触摸失效

但是微软发现在书写时，如果走 Touch 消息，此时因为需要经过 Windows 消息，速度没有之前从硬件获取的快，于是提出了 Windows Inking Service Platform 请看 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

在 Win7 的这个触摸架构就是古代的触摸

很多开发者都会反馈在 Win7 开发的触摸失效问题，如 [WPF 客户端开发需要知道的触摸失效问题](https://blog.lindexi.com/post/WPF-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BC%80%E5%8F%91%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88%E9%97%AE%E9%A2%98.html) 和需要区分 Touch 和 Stylus 和鼠标事件，这样的开发成本比较高

在 Windows 8 提出了现代的触摸方法，通过 Pointer 表示，无论是鼠标还是触摸还是笔都是使用相同的 Pointer 消息，这个触摸架构将会对触摸有很好的支持。默认在 UWP 应用使用这个触摸架构，而 WPF 程序可以通过 [这个方法](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html ) 开启，对于 win32 应用需要使用 [EnableMouseInPointer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-enablemouseinpointer ) 方法在进程内开启

在上古的系统，因为触摸屏没有大量使用，也没有标准，只能各个应用读写硬件设备自己做触摸。在 XP 的时候，开始引入触摸屏 HID 设备标准（不确定win98又没引入，毕竟我不是学历史的） 此时触摸刚起步，存在很多不合理的设计，多点触摸还没有完善，需要依赖额外的驱动和补丁。在 Win7 将触摸作为 Windows 消息，支持了多点的触摸，同时兼容 XP 和上古的和硬件设备读写的软件，但是 Win7 系统消息的转发受补丁和驱动的影响，也会存在一些触摸失效，同时因为 Touch 消息和鼠标消息分开，降低了开发效率。在 Win8 提出了 Pointer 消息，可以将触摸等合并为相同的消息

本文主要的是下面文档，更多细节请看文档

[Introduction to WPF 4 Multitouch – Jaime Rodriguez](https://blogs.msdn.microsoft.com/jaimer/2009/11/04/introduction-to-wpf-4-multitouch/ )

[c# - WPF supports touch or multi-touch screen? - Stack Overflow](https://stackoverflow.com/questions/1254616/wpf-supports-touch-or-multi-touch-screen )

[MultiTouch driver for Windows XP Tablet PC Edition 2005 - ThinkPad X60 Tablet, X61 Tablet, X200 Tablet - PL](https://support.lenovo.com/pl/en/downloads/migr-67061 )

[Getting Started with Windows Touch Messages](https://docs.microsoft.com/en-us/windows/win32/wintouch/getting-started-with-multi-touch-messages )

[Getting Started with Windows Touch Gestures](https://docs.microsoft.com/en-us/windows/win32/wintouch/getting-started-with-multi-touch-gestures )

[TranslateZoomRotateBehavior](https://docs.microsoft.com/en-us/previous-versions/visualstudio/design-tools/expression-studio-4/ff723978(v=expression.40)?redirectedfrom=MSDN )

[Pointer Input Messages and Notifications](https://docs.microsoft.com/en-us/windows/win32/api/_inputmsg/ )

[win10 支持默认把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html )

[WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

[WPF 客户端开发需要知道的触摸失效问题](https://blog.lindexi.com/post/WPF-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BC%80%E5%8F%91%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88%E9%97%AE%E9%A2%98.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

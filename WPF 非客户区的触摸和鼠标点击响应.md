# WPF 非客户区的触摸和鼠标点击响应

默认在 WPF 里面是不响应非客户区的鼠标事件，但响应触摸事件

<!--more-->
<!-- CreateTime:2019/11/29 8:44:11 -->

<!-- csdn -->

在没有喝下午茶的时候 [lsj](https://blog.sdlsj.net/ ) 告诉我，在项目里面在一个定制的窗口里面的非客户区用鼠标点击不了一个按钮，但是用触摸可以点击按钮。本金鱼一开始认为这是之前修复的问题，但是作为金鱼已经不记得是怎么修了，为了让本金鱼下次遇到触摸或鼠标问题的时候可以解决，于是写了这个博客

本文将会告诉大家在 WPF 里面关于非客户区的触摸和鼠标点击响应

在本文开始之前，需要大家知道非客户区 Non-client Area 的概念，其实就是窗口标题栏大概的意思，详细请看 [一起学WPF系列（3）：窗体 - Robin Zhang - 博客园](https://www.cnblogs.com/jillzhang/archive/2008/04/05/1138526.html ) 和 [WPF 使用 WindowChrome，在自定义窗口标题栏的同时最大程度保留原生窗口样式（类似 UWP/Chrome） - walterlv](https://blog.walterlv.com/post/wpf-simulate-native-window-style-using-window-chrome.html )

敲黑板，下面的知识点要考

默认的 WPF 程序支持在非客户区响应 Touch 触摸，但不响应鼠标点击和 Pointer 触摸

如果需要在非客户区也就是窗口标题栏支持鼠标点击，那么请在按钮添加附加属性 WindowChrome.IsHitTestVisibleInChrome 为 true 支持鼠标点击

```csharp
<Button WindowChrome.IsHitTestVisibleInChrome="True"/>
```

这里的 `WindowChrome.IsHitTestVisibleInChrome` 支持继承，也就是可以在窗口设置，这样所有在窗口里面的按钮都支持在标题栏点击

如何开启 Pointer 消息请看 [win10 支持默认把触摸提升鼠标事件 打开 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html )

我和 [lsj](https://blog.sdlsj.net/ ) 使用 spy++ 知道在 WPF 的标题栏点击的时候是可以收到 Windows 鼠标消息，也就是这里是 WPF 处理的

我和 [lsj](https://blog.sdlsj.net/ ) 说也许是之前的 WPF 框架的大佬写的时候还不知道有触摸，于是处理了鼠标事件。之后添加了触摸忘了需要去掉，于是触摸就可以使用。现在 [lsj](https://blog.sdlsj.net/ ) 正在看 WPF 的源代码，想要找到是如何让标题栏支持触摸但是不支持鼠标点击

[WindowChrome Class (System.Windows.Shell)](https://docs.microsoft.com/en-us/dotnet/api/system.windows.shell.windowchrome?wt.mc_id=MVP )

[WindowChrome.IsHitTestVisibleInChrome Attached Property (Microsoft.Windows.Shell)](https://docs.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/ff702336(v%3Dvs.100) )

[关于WM_NCHITTEST消息 - Gang.Wang - 博客园](https://www.cnblogs.com/GnagWang/archive/2010/09/12/1824394.html )

[WM_TOUCH message - Windows applications](https://docs.microsoft.com/en-us/windows/win32/wintouch/wm-touchdown )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

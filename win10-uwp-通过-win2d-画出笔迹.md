
# win10 uwp 通过 win2d 画出笔迹

本文告诉大家如何在 UWP 上让 win2d 画出笔迹，通过实际测试发现在 UWP 的笔迹的性能比在 WPF 高很多。但是如果只是使用默认的 InkCanvas 可以做的很少，同时性能也不是特别高，在加上 win2d 才可以做到和来画一样快的性能

<!--more-->


<!-- csdn -->
<!-- 标签：uwp,win2d -->

在参加[微软技术暨生态大会 2018](https://walterlv.gitee.io/post/tech-summit-2018.html )听了[邵猛](https://www.cnblogs.com/shaomeng/archive/2018/01/14/8228944.html )大佬的[利用 Windows 新特性开发出更好的手绘视频应用](https://www.cnblogs.com/shaomeng/p/9769270.html )学到了使用 win2d 可以画出笔迹。

在之前我一直在想来画的笔迹性能为什么那么好，现在终于了解到了，于是本文就将具体实现写出来。本文的代码不可以用在实际项目上，因为假设用户都是正常书写

在 UWP 的笔迹有设置对笔迹完全控制，在中文翻译，会将 Ink 翻译为墨迹，本文将 Ink 翻译为笔迹或墨迹。

## 界面

如果想要在 win2d 画出笔迹，还是需要使用 InkCanvas 来收集笔迹，不能直接通过 Pointer 来做。通过测试使用 Pointer 和 InkCanvas 的性能相差在我的设备是 16 ms 左右，需要知道，笔迹的书写过程，相差 16 ms 是一个很大的值。

至于为什么通过 InkCanvas 收集笔迹需要在本文下方告诉大家 InkCanvas 的原理。

因为使用 win2d 需要通过 Nuget 安装，这部分请看[在项目安装win2d](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html) 本文就直接使用

先引用命名空间 `xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"` 这样就可以在界面通过 canvas 使用高性能的 win2d 来画笔迹







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
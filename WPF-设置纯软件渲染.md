
# WPF 设置纯软件渲染

最近看到有小伙伴说 WPF 使用硬件渲染，如何让 WPF 不使用硬件渲染，因为他觉得性能太好了。万一这个版本发布了，产品经理说下个版本要提升性能就不好了。于是就找到一个快速的方法，让程序不使用硬件渲染这样下个版本要优化就让程序使用硬件渲染。

<!--more-->


<!-- csdn -->

设置 WPF 使用软件渲染的方法是在 .net framework 3.5 之后才可以的。使用方法很简单，在 Loaded 之后，添加下面代码

```csharp
                HwndSource hwndSource = PresentationSource.FromVisual(this) as HwndSource;
                HwndTarget hwndTarget = hwndSource.CompositionTarget;
                hwndTarget.RenderMode = RenderMode.SoftwareOnly;
```

默认的 RenderMode 是 默认，也就是如果判断有硬件就在硬件渲染，如果没有就在 CPU 渲染。

如果设置 SoftwareOnly 就不在硬件渲染。

除了想降低性能，估计没有人会设置这个。

最近在做渲染优化，更多博客请看 [http://lindexi.gitee.io](http://lindexi.gitee.io )

[WPF 渲染级别](https://lindexi.oschina.io/lindexi/post/WPF-%E6%B8%B2%E6%9F%93%E7%BA%A7%E5%88%AB.html )

[WPF 使用 Direct2D1 画图入门](https://lindexi.oschina.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
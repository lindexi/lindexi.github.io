# wpf 使用 Dispatcher.Invoke 冻结窗口

如果使用`Dispatcher.Invoke`实际上会有一个坑，在执行`Dispatcher.Invoke`刚好拖动窗口就会出现窗口冻结，这时使用 Alt+Tab 可以解决。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<div id="toc"></div>

<!-- csdn -->

<!-- 标签：wpf,性能优化 -->

这个问题是在我写[wpf DoEvents](https://lindexi.oschina.io/lindexi/post/wpf-DoEvents.html )发现的，因为`Dispatcher.Invoke`可以让界面刷新，但是在拖动窗口会让窗口冻结。

所以一个建议的方法是使用`Dispatcher.InvokeAsync` ，如果需要深入了解，请看我师傅的文章[深入了解 WPF Dispatcher 的工作原理](https://walterlv.github.io/post/dotnet/2017/09/26/dispatcher-invoke-async.html )

在所有使用`Dispatcher.Invoke`的代码都可以通过使用`await Dispatcher.InvokeAsync`去替换。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# WPF 获得触笔悬停元素上

触笔可以获得悬停在元素上，这时触笔没有碰到元素，没有碰到屏幕。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<!-- csdn -->

如果使用触笔，那么在悬停就需要显示光标位置，这时使用`UIElement.StylusInAirMove` 事件可以获得触笔悬停在元素上。

需要知道，这个事件是 .net Framework 3.0 之后添加的。

[UIElement.StylusInAirMove 事件 (System.Windows)](https://msdn.microsoft.com/zh-cn/library/system.windows.uielement.stylusinairmove(v=vs.110).aspx )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
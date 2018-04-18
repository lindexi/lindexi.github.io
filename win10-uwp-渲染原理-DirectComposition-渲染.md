
# win10 uwp 渲染原理 DirectComposition 渲染

本文来告诉大家一个新的技术[DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx )，在 win7 之后（实际上是 vista），微软正在考虑一个新的渲染机制。

<!--more-->


<!-- csdn -->
<!-- 草稿 -->

在 Windows  Vista 就引入了一个服务，桌面窗口管理器[Desktop Window Manager](https://msdn.microsoft.com/en-us/library/windows/desktop/aa969540(v=vs.85).aspx )，虽然从[借助 C++ 进行 Windows 开发](https://msdn.microsoft.com/magazine/dn745861 )博客可以看到 DWM 不是一个好的方法，但是比之前好。

在 win8 的时候，微软提出了 DirectComposition ，这是一个新的方法。

在软件的渲染一直都是两个阵营，一个是使用直接渲染模式。直接渲染的例子是使用 Direct2D 和 Direct3D ，而直接通过 Dx api 的方式当然需要使用 C++ 和底层的 API ，这开发效率比较差。



参考：

[借助 C++ 进行 Windows 开发 - 使用 Windows 组合引擎实现高性能窗口分层](https://msdn.microsoft.com/magazine/dn745861 )

[借助 C++ 进行 Windows 开发 - 使用 Windows 组合引擎](https://msdn.microsoft.com/magazine/dn786854 )

[Windows, UI and Composition (the Visual Layer) – Mike Taulty](https://mtaulty.com/2015/12/17/m_15996/ )

[Windows with C++ - DirectComposition: A Retained-Mode API to Rule Them All](https://msdn.microsoft.com/en-us/magazine/dn759437.aspx )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
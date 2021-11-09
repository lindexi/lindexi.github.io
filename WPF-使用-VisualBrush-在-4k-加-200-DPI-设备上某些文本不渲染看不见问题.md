
# WPF 使用 VisualBrush 在 4k 加 200 DPI 设备上某些文本不渲染看不见问题

这是我做一个十万点实时刷新的图表控件遇到的问题，做过高性能图表的伙伴大概都知道，此时需要关闭命中测试的功能，无论是控件的还是 Drawing 的，否则计算命中测试的耗时将会让主线程卡住。为了解决此问题，有多个可以选择的方法，在此控件，我选择的是采用 VisualBrush 的方法。将 DrawingVisual 绘制到 VisualBrush 里面，再将 VisualBrush 作为贴图给矩形使用，这样的优势在于可以在命中测试的时候，只处理矩形。矩形命中测试的耗时可以忽略。但是在一些 4k 加百分之 200 的 DPI 缩放设备上，看不到某些 GlyphRun 的内容，本文记录此问题和对应的解决方法

<!--more-->



<!-- 草稿 -->

前置要求：

- 4k 分辨率屏幕
- 百分之两百 DPI 缩放
- 使用 GlyphRun 直接或间接
- 绘制到 VisualBrush 中

在 WPF 的底层文本绘制都是采用 GlyphRun 绘制，因此可以认定为影响为全部文本，以及对应的文本控件

现象：

有某些文本内容不绘制渲染出来，看不见某些文本内容，但是在相同的 DrawingContext 里面的其他绘制内容，如线条或图片等都可以正常绘制出来

以上的现象包括：

- 在某些设备上，暂时未找到具体影响因素
- 某些文本内容不可见，而不是全部文本内容
- 对整个控件进行 RenderTransform 之后可以让某些文本可见
- 对界面进行刷新，可以让文本可见
- 对界面进行偶数次刷新，文本不可见






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
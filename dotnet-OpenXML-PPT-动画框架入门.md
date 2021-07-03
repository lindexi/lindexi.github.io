
# dotnet OpenXML PPT 动画框架入门

本文将从 OpenXML 方面聊 PPT 的动画框架，本文是属于编程方面而不是 PPT 动画制作教程

<!--more-->


<!-- CreateTime:2021/7/2 16:03:31 -->


<!-- 草稿 -->

开始之前，还请掌握一些基础知识，如阅读以下博客

- [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html)
- [Office 文档解析 文档格式和协议](https://blog.lindexi.com/post/Office-%E6%96%87%E6%A1%A3%E8%A7%A3%E6%9E%90-%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F%E5%92%8C%E5%8D%8F%E8%AE%AE.html )
- [dotnet OpenXML 解析 PPT 页面元素文档格式](https://blog.lindexi.com/post/dotnet-OpenXML-%E8%A7%A3%E6%9E%90-PPT-%E9%A1%B5%E9%9D%A2%E5%85%83%E7%B4%A0%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F.html )

本文不讨论 Slide Master 和 Slide Layout 的动画，关于这两个请参阅 [dotnet OpenXML 的 Slide Master 和 Slide Layout 是什么](https://blog.lindexi.com/post/dotnet-OpenXML-%E7%9A%84-Slide-Master-%E5%92%8C-Slide-Layout-%E6%98%AF%E4%BB%80%E4%B9%88.html )

本文只讨论 Slide 页面里面的动画

## 元素主序列动画

在 OpenXML 中，如果一个动画是依靠翻页或点击页面进行触发的，那么这些动画有

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
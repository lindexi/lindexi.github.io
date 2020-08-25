
# dotnet OpenXML 文本 Kerning 字间距的作用

在 PPT 和 Word 排版里面，小伙伴会接触到 Kerning 字间距这个属性，本文将告诉大家这个属性的功能，以及为什么需要在 OpenXml 里面设置这个属性

<!--more-->


<!-- CreateTime:2020/8/24 8:47:14 -->


<!-- 发布 -->

其实这个属性的实际作用需要从文本排版知识开始说起，在 Latin 拉丁语，如英文，这些语言的文本排版惯例里面，因为是由多个独立的字符构建的单词。因此多个字符之间的距离，也就是本文说的 Kerning 字间距将会影响阅读者的阅读效率。甚至听说（无依据）古老的时候印刷社有这样一个职业，决定每个单词的每个字符的间距，让读者看起来爽

在当前数字化的时代，自然做排版软件也需要考虑这一点，如在排版 AVATAR 这个经典单词的时候，将会发现如 Word 等成熟的排版软件，在微软雅黑字体下将会更改每个字符的默认间距，而且是依据前后字符的不同而不同

<!-- ![](image/dotnet OpenXML 文本 Kerning 字间距的作用/dotnet OpenXML 文本 Kerning 字间距的作用0.png) -->

![](http://image.acmx.xyz/lindexi%2Fdotnet%2520OpenXML%2520%25E6%2596%2587%25E6%259C%25AC%2520Kerning%2520%25E5%25AD%2597%25E9%2597%25B4%25E8%25B7%259D%25E7%259A%2584%25E4%25BD%259C%25E7%2594%25A80.png)

在第一行是经过优化的文本，可以看到 AV 这两个字符的间距和 AR 是不相同，而 AV 和 AT 都会有两个字符重叠，此时在英语语系阅读第一行字符会感到舒服

而第二行就是一些简单排版软件，这些软件每个字符都是独立排版的，没有根据前后的字符决定字间距，此时的排版看起来没有第一行好

而在 OpenXML 的 Kerning 是可以让排版工作者根据需要修改默认的字体间距的功能。因为排版软件有时候也会猜错规则，如有时候需要表达一些特殊的词，这些词很多都是宗教相关，此时需要修改字间距，让字符比较紧等

而字间距的值是由字体表决定的，这将会根据不同的字体而不同，因此想要做到和 Word 一样强的排版，就需要解析字体表，同时了解语言规则

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
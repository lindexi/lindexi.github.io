# WPF 修改图片颜色

本文告诉大家如何修改图片的颜色，如去掉图片的蓝色

<!-- 标签：WPF，图片处理 -->

<!--more-->
<!-- csdn -->

<!-- 草稿 -->

在 WPF 可以使用很多图片处理的方法，本文告诉大家的是一个图片处理，可以把处理的图片保存在文件。

在阅读本文，我假设大家是熟悉 WPF 的，至少了解 C# ，也知道图片的格式。

在 WPF 可以使用 ARBG 数组表示图片，本文修改图片颜色的方法就是使用 ARBG 数组的方法修改，修改里面的元素的值。

如我需要去掉图片的蓝色，就可以通过修改 ARBG 数组的元素，设置所有蓝色为 0 ，去掉蓝色。

读取图片，把

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

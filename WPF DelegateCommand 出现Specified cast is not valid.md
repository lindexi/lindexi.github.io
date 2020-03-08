# WPF DelegateCommand 出现Specified cast is not valid

使用 DelegateCommand 出现 Specified cast is not valid

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


最近写快捷键需要 DelegateCommand ，于是用了 `DelegateCommand<double> ` ，运行时出现 Specified cast is not valid 

原因是 DelegateCommand 传入的 Object 是可空的，如果使用 Double ，那么是不可空的，就出现错误

简单的方法是用 double?

于是就可以啦

如果遇到 DelegateCommand 出现这个错误，一般就是使用不可空的类型，只要让他可空就好。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
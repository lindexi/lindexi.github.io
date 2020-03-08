# win10 uwp 从Type使用构造

本文主要：如何从 Type new一个对象

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

原本从 WPF ，要想 new 一个对象从 type ，可以使用`type.Assembly.CreateInstance(type.FullName);`

但是在 UWP ，需要使用`type.GetConstructor(Type.EmptyTypes).Invoke(parameters);`

多谢 durow ，找了很久在他写的 http://www.cnblogs.com/durow/p/4883556.html 刚好有这个

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
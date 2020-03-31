
# Xamarin 使用 GTK 提示找不到 libglib-2.0-0.dll 找不到

在使用 Xamarin 开发 Linux 应用的时候，刚开始如果没有弄好 libglib-2.0-0.dll 的依赖库，那么将会在运行的时候，在 Gtk.Application.Init() 这句代码提示找不到这个库

<!--more-->


<!-- 发布 -->

解决方法是先到[官网](https://www.monodevelop.com/download/#fndtn-download-win) 下载 `GTK#` 安装包或 mono x86 的应用

安装到默认路径，也就是在 `C:\Program Files (x86)\GtkSharp\2.12\bin` 路径，默认安装的时候会加入到环境变量

接下来到 `C:\Program Files (x86)\GtkSharp\2.12\bin` 复制 libglib-2.0-0.dll 文件到 xamarin 的输出文件夹，如 `D:\lindexi\t\Xamarin\Cla\bin\x86\Debug\net47` 文件夹里面，此时尝试运行，应该就不会存在这个提示

注意现在 GTK# 仅支持 x86 应用





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
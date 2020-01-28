# 在 SublimeText 使用 dotnet 编译 C# 项目

在 SublimeText 搭建 C# 环境可以找到的博客基本都是使用 csc 进行构建，而我期望在 dotnet 下编译整个项目。通过 dotnet 编译整个项目可以解决编译大项目时需要打开一个控制台降低效率

<!--more-->
<!-- 发布 -->

用 dotnet 编译的优点是我可以在 Ubuntu 系统使用 SublimeText 编写和编译 C# 项目。我最近无聊弄了一个 Ubuntu 系统在玩，在 Ubuntu 系统下确实需要缺啥写啥，如果不是要玩，还是推荐不要用这个系统。因为我还不熟悉这个系统，用的效率特别低。例如我想写一个 C# 程序，我想要在 SublimeText 通过 `ctrl+B` 进行编译然后运行，而原本在 Windows 下我可以同步我的配置，在这里就不能使用，原因是在 Windows 下通过 csc 编译文件



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

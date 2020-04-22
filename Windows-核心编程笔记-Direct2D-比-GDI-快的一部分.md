
# Windows 核心编程笔记 Direct2D 比 GDI 快的一部分

每次看圣经都有不同的收获，在阅读了内核相关的内容，发现了有从用户模式和内核模式切换的角度说明 Direct2D 比 GDI 快的一部分。但这不是说明 Direct2D 一定比 GDI 快，而是在应用程序消费在用户模式和内核模式切换的耗时这一部分是 Direct2D 更少

<!--more-->


<!-- CreateTime:4/21/2020 8:47:14 AM -->


对于一个用户线程来说，有一部分时间在用户模式下运行，另一部分时间在内核模式下运行。而图形和窗口处理是在内核模式处理，也就是大量使用 GDI 的密集图形的应用将会频繁切换用户模式和内核模式。而 Direct2D 是在用户模式执行大量的计算，只是单纯将画面数据发送到内核模式进行合成，减少用户模式和内核模式切换所花费的时间

可以使用性能计数器证明上面这一点







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
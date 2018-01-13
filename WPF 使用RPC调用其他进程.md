# WPF 使用RPC调用其他进程

如果在 WPF 需要用多进程通信，一个推荐的方法是 WCF ，因为 WCF 是 RPC 计算。先来讲下 RPC (Remote Procedure Call) 远程过程调用，他是通过特定协议，包括 tcp 、http 等对其他进程进行调用的技术。详细请看[百度](https://baike.baidu.com/item/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8%E5%8D%8F%E8%AE%AE?fromtitle=RPC&fromid=609861)

<!--more-->
<!-- csdn -->

现在不会告诉大家如何使用 WCF ，下面讲的是使用 remoting 这个方法。需要知道 dotnet remoting 是已经过时的技术，建议使用 wcf 但是 wcf 部署难度比较高，对于性能要求比较高或想快速使用，建议使用 remoting 。使用方法很简单



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

# dotnet remoting 的 Channel 详解

本文详细告诉大家在 dotnet remoting 的各个 Channel 是做什么的。

<!--more-->
<!-- CreateTime:2018/9/2 14:45:41 -->

<!-- csdn -->

<!-- 草稿 -->

<!-- 标签：remoting，WPF,dotnetremoting,rpc,多进程 -->

本文基于控制台和 WPF 程序作为例子告诉大家三个可以选的 Channel 各自的优点缺点。

在开始本文之前，假设大家已经初步理解 dotnet remoting 了，如果还没有开始接触 dotnet remoting 推荐大家阅读我的博客 [WPF 使用RPC调用其他进程 - CSDN博客](https://blog.csdn.net/lindexi_gd/article/details/80373135 )

在开始之前需要说明什么是 Channel ，从 Channel 的命名可以知道，这个类是用在传输的过程。在 dotnet remoting 就是需要通过某个方式联系多个进程。大家可以想到连接的方法可以通过网络或管道。

是的，在 dotnet remoting 可以使用三个不同的 Channel 就是基于网络和管道。其中有基于 tcp 的 TcpChannel 和 基于 http 的 HTTPChannel 最后还有基于管道的 IPCChannel 三个。

在 dotnet framework 2.0 之后才可以使用 IPCChannel 这个通信是通过管道来做到的。适合在相同的设备运行不同进程的通信。

从字面就可以理解 TCPChannel 和 HTTPChannel 的作用，所以这里就不需要多说，所有的 Channel 都有共同的基类使用上也都是相同的。只是选用不同的 Channel 有不同的用途。

如 HTTPChannel 可以用在和 IIS 的交互上，通过 HTTPChannel 可以让软件很通用。

通过 TCPChannel 可以做到对网络环境要求比较高的程序，因为 TCPChannel 可以做到比 HTTPChannel 更快。

如果只是在相同的设备运行，而且不想要关注端口，就建议使用 IPCChannel 通过这个方式可以创建任意的命名作为端口，而且不经过网络。也就不存在网络的问题，而且在相同的设备运行的性能是最快的。

可以使用 .Net Remoting 的包括：

 - 控制台

 - WinForm 程序

 - WPF 程序

 - IIS 程序

现在除非使用 HTTPChannel 和 TCPChannel 而且自己写解析不然无法在 dotnet core 使用。


参见：

[WPF 使用RPC调用其他进程 - CSDN博客](https://blog.csdn.net/lindexi_gd/article/details/80373135 )

《Professional C#》 第3版

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

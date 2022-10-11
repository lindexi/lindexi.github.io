
# 解决 System.Net.Sockets.SocketException 10045 参考的对象类型不支持尝试的操作 无法联网

本文收集 System.Net.Sockets.SocketException 异常错误码为 10045 导致无法联网的问题

<!--more-->


<!-- 发布 -->

这里的 10045 是 Win32 的 Socket 错误码，可以从 [Windows Sockets Error Codes (Winsock2.h) - Win32 apps Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/winsock/windows-sockets-error-codes-2 ) 文档了解到 10045 对应的是 WSAEOPNOTSUPP 错误，描述如下

> Operation not supported.
>   The attempted operation is not supported for the type of object referenced. Usually this occurs when a socket descriptor to a socket that cannot support this operation is trying to accept a connection on a datagram socket.

这一层出现问题，和 .NET 的关系不大，其他技术也可能会遇到相同的问题

先去注册表的 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters\Protocol_Catalog9` 路径里面，在各个子路径，找找有没有奇怪的 DLL 将其删除，然后再尝试重置网络重启电脑。如果只是想查看而已，也可以通过命令行输入 `netsh winsock show catalog` 显示所有的文件

解决方法：

命令行输入以下代码，然后重启机器即可

```
netsh winsock reset
```

更多请看 [解决 System.Net.Sockets.SocketException 10106 无法加载或初始化请求的服务提供程序 无法联网](https://blog.lindexi.com/post/%E8%A7%A3%E5%86%B3-System.Net.Sockets.SocketException-10106-%E6%97%A0%E6%B3%95%E5%8A%A0%E8%BD%BD%E6%88%96%E5%88%9D%E5%A7%8B%E5%8C%96%E8%AF%B7%E6%B1%82%E7%9A%84%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E7%A8%8B%E5%BA%8F-%E6%97%A0%E6%B3%95%E8%81%94%E7%BD%91.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
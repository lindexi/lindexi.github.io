# 解决 System.Net.Sockets.SocketException 10106 无法加载或初始化请求的服务提供程序 无法联网

本文收集 System.Net.Sockets.SocketException 异常错误码为 10106 导致无法联网的问题

<!--more-->
<!-- CreateTime:2022/9/1 19:30:56 -->

<!-- 发布 -->

这里的 10106 是 Win32 的 Socket 错误码，可以从 [Windows Sockets Error Codes (Winsock2.h) - Win32 apps Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/winsock/windows-sockets-error-codes-2 ) 文档了解到 10106 对应的是 WSAEPROVIDERFAILEDINIT 错误，描述如下

> Service provider failed to initialize.
>   The requested service provider could not be loaded or initialized. This error is returned if either a service provider's DLL could not be loaded (LoadLibrary failed) or the provider's WSPStartup or NSPStartup function failed.

这一层出现问题，和 .NET 的关系不大，其他技术也可能会遇到相同的问题

解决方法：

命令行输入以下代码，然后重启机器即可

```
netsh winsock reset
```

相关参考博客：

- [System.Net.Sockets.SocketException: 无法加载或初始化请求的服务提供程序 - 二笔青年](https://blog.clso.fun/posts/2019-09-29/137.html )
- [socket 10106问题解决日记_忆常的博客-CSDN博客](https://blog.csdn.net/wjtxt/article/details/10500817 )
- [服务器上tcp程序突然连接不上报错 System.Net.Sockets.SocketException (0x80004005): 无法加载或初始化请求的服务提供程序。 - Ace001 - 博客园](https://www.cnblogs.com/xuejianxiyang/p/14512955.html )

# dotnet 已知问题 NamedPipeClientStream 连接不存在的服务在内部抛出异常

本文记录一个 dotnet 已知问题，此问题在 dotnet 9 之前就存在。在 Linux 系统上，使用 NamedPipeClientStream 连接不存在的服务时，将不断疯狂地抛出 SocketException 异常

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

问题表现：

使用 NamedPipeClientStream 的 Connect 方法连接一个不存在的管道名时，将会发现在 dotnet 底层疯狂抛出 System.Net.Sockets.SocketException (99): Cannot assign requested address /tmp/CoreFxPipe_Xxx 异常

最简复现代码如下，可到本文末尾找到所有代码的下载方法

```csharp
var namedPipeClientStream = new NamedPipeClientStream("DoNotExistFoo");
namedPipeClientStream.Connect();
```

由于这个异常只发生在 dotnet 内部，被 dotnet 内部进行捕获，想要更好看到这个异常，则需要监听 `AppDomain.CurrentDomain.FirstChanceException` 事件，修改之后的代码如下

```csharp
using System.IO.Pipes;

AppDomain.CurrentDomain.FirstChanceException += (sender, eventArgs) =>
{
    Console.WriteLine($"FirstChanceException: {eventArgs.Exception.GetType().FullName} {eventArgs.Exception.Message}");
};

var namedPipeClientStream = new NamedPipeClientStream("DoNotExistFoo");
namedPipeClientStream.Connect();
```

尝试将此代码放在 Linux 机器上运行，如 UOS 或麒麟系统上，将可以看到控制台刷屏，可见异常输出信息如下

```
System.Net.Sockets.SocketException (99): Cannot assign requested address /tmp/CoreFxPipe_DoNotExistFoo
   at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
FirstChanceException: System.Net.Sockets.SocketException (99): Cannot assign requested address /tmp/CoreFxPipe_DoNotExistFoo
```

具体堆栈如下

```
  System.Net.Sockets.dll!System.Net.Sockets.Socket.DoConnect(System.Net.EndPoint endPointSnapshot, System.Net.SocketAddress socketAddress)  C#
  System.Net.Sockets.dll!System.Net.Sockets.Socket.Connect(System.Net.EndPoint remoteEP)  C#
  System.IO.Pipes.dll!System.IO.Pipes.NamedPipeClientStream.TryConnect(int _) C#
  System.IO.Pipes.dll!System.IO.Pipes.NamedPipeClientStream.ConnectInternal(int timeout = -1, System.Threading.CancellationToken cancellationToken, int startTime = 542050485)  C#
```

此问题可能很难从底层改掉，业务层优化会更加简单，那就是在 Connect 方法里面传递限制时间，防止无限等待

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/31318b24ff8656a05f0f81bef867b501e425d6bf/Workbench/BeekefanereHawgudairwochemwhi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/31318b24ff8656a05f0f81bef867b501e425d6bf/Workbench/BeekefanereHawgudairwochemwhi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 31318b24ff8656a05f0f81bef867b501e425d6bf
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 31318b24ff8656a05f0f81bef867b501e425d6bf
```

获取代码之后，进入 Workbench/BeekefanereHawgudairwochemwhi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
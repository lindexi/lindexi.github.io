
# dotnet 6 精细控制 HttpClient 网络请求超时

本文告诉大家如何在 dotnet 6 下使用 HttpClient 更加精细的控制网络请求的超时，实现 HttpWebRequest 的 ReadWriteTimeout 功能

<!--more-->


<!-- CreateTime:2022/7/25 8:28:00 -->

<!-- 发布 -->
<!-- 博客 -->

本文将介绍如何在 HttpClient 控制以下网络行为的超时

- 网络连接超时
- 网络请求超时
- 网络响应超时
- 网络总超时

在 dotnet 6 下 HttpClient 只是一个包装类，实际的网络请求的核心实现是通过 SocketsHttpHandler 实现的，详细请看 [dotnet 6 HttpClientHandler 和 SocketsHttpHandler 有什么差别 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/16492703.html)

在 HttpClient 里，由于 HttpClient 自带的 Timeout 碰触不到底层网络，导致了 Timeout 属性控制范围太广，很多业务上都不合适使用，比如做大文件上传，自然在上传过程中就超时了，如果用户的网络上传速度不快。在 HttpClient 里面，设置 Timeout 表示设置整个网络请求过程的总超时时间。如果只是期望设置连接超时，那自然是做不到的

既然实际的网络是 SocketsHttpHandler 实现的，在 SocketsHttpHandler 可以进行更加精细的控制，例如通过 ConnectTimeout 属性即可用来控制连接的超时时间

```csharp
        var handler = new SocketsHttpHandler()
        {
            ConnectTimeout = TimeSpan.FromSeconds(10),
        };
        var client = new HttpClient(handler);
```

这里值得敲黑板的是在 dotnet 6 下，将会大量的复用连接，也就是如果不逗比的情况下，多次对相同的链接请求，在时间距离不远的前提下，是可以复用连接的，不需要做重复的连接。特别是在设置 SocketsHttpHandler 的 EnableMultipleHttp2Connections 为 true 再加上服务器端也支持 Http 2 的多路复用情况下

如果是想和 HttpWebRequest 一样控制 ReadWriteTimeout 的时间，在 dotnet 6 下，可以对请求和响应，也就是发送和接收做分别的超时控制，这就是用到了 dotnet 6 新的 ConnectCallback 属性实现，例子代码如下

```csharp
        handler.ConnectCallback = async (context, cancellationToken) =>
        {
            var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);

            try
            {
                socket.NoDelay = true;

                // 这里可以自己偷偷改掉域名哦，也就是将原本请求的域名修改为一个奇怪的域名。这里偷偷改了，团队的其他伙伴可是很难调试出来的哦
                await socket.ConnectAsync(context.DnsEndPoint, cancellationToken)
                // 配置异步等待后不需要回到原来的线程
                .ConfigureAwait(false);

                // 发送的超时时间，相当于请求的超时
                socket.SendTimeout = (int) TimeSpan.FromSeconds(10).TotalMilliseconds;
                // 接收的超时时间，相当于响应的超时
                socket.ReceiveTimeout = (int) TimeSpan.FromSeconds(5).TotalMilliseconds;
            }
            catch
            {
                socket.Dispose();
                throw;
            }

            // 在 NetworkStream 里，设置 ownsSocket 参数为 true 将会在 NetworkStream 被释放的时候，自动释放 Socket 资源
            return new NetworkStream(socket, ownsSocket: true);
        };
```

可以看到 HttpClient 的控制是比 HttpWebRequest 更强的，可以分别控制请求和响应的超时

另外，这里的 ConnectCallback 也如上文描述，由于 HttpClient 将会尽可能复用连接，不一定每次请求都会进来，建议不要将配置作为动态配置，想要根据业务动态决定超时时间是不靠谱的行为，这里应该是初始化过程，给定准确的值

回顾一下，控制网络总超时，使用 HttpClient 自带的 Timeout 属性

控制网络的连接超时，使用 SocketsHttpHandler 的 ConnectTimeout 属性

控制网络的请求超时，使用 Socket 的 SendTimeout 属性

控制网络的响应超时，使用 Socket 的 ReceiveTimeout 属性

更多请参阅 [dotnet 6 使用 HttpClient 的超时机制](https://blog.lindexi.com/post/dotnet-6-%E4%BD%BF%E7%94%A8-HttpClient-%E7%9A%84%E8%B6%85%E6%97%B6%E6%9C%BA%E5%88%B6.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# dotnet 6 HttpClientHandler 和 SocketsHttpHandler 有什么差别

本文来告诉大家在 dotnet 6 的 HttpClientHandler 和 SocketsHttpHandler 两个类型有什么不同

<!--more-->
<!-- CreateTime:2022/6/16 10:01:22 -->

<!-- 发布 -->
<!-- 博客 -->

在创建 HttpClient 时，可以在 HttpClient 的构造函数传入 HttpMessageHandler 类型的参数，此参数将执行实际的逻辑。其中常用的传入参数类型就是 HttpClientHandler 和 SocketsHttpHandler 类型

那这两个类型有什么差别呢？根据[官方文档](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?WT.mc_id=WD-MVP-5003260) 可以了解到，从 .NET Core 2.1 开始，默认的 HttpClient 底层的网络通讯实现就是靠 [System.Net.Http.SocketsHttpHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.socketshttphandler?WT.mc_id=WD-MVP-5003260) 实现的，替代了原先的 HttpClientHandler 类型

也就是说在 dotnet 6 采用的 HttpClient 底层使用的是 SocketsHttpHandler 类型作为默认的网络通讯。那原有的使用 HttpClientHandler 的代码呢？其实在底层也做了统一，使用 HttpClientHandler 也将在底层采用 SocketsHttpHandler 作为网络通讯

请看 dotnet 6 的 HttpClientHandler 的源代码

```csharp
using HttpHandlerType = System.Net.Http.SocketsHttpHandler;

    public partial class HttpClientHandler : HttpMessageHandler
    {
        private readonly HttpHandlerType _underlyingHandler;

        private HttpMessageHandler Handler
            => _underlyingHandler;

        public HttpClientHandler()
        {
            _underlyingHandler = new HttpHandlerType();

            ClientCertificateOptions = ClientCertificateOption.Manual;
        }

        protected internal override HttpResponseMessage Send(HttpRequestMessage request, CancellationToken cancellationToken) =>
            Handler.Send(request, cancellationToken);

        protected internal override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken) =>
            Handler.SendAsync(request, cancellationToken);

        // 忽略其他逻辑
    }
```

可以看到在 HttpClientHandler 的底层实现就是使用 SocketsHttpHandler 来实现的

也就是说，无论你在 HttpClient 传入的参数类型是 SocketsHttpHandler 还是 HttpClientHandler 类型，在 dotnet 6 都会实际上调用到 SocketsHttpHandler 类型。还请不需要纠结这部分的差异

任何对 HttpClientHandler 的配置都会被设置到底层的 SocketsHttpHandler 类型的 Handler 属性

唯一需要开始纠结差异的部分只是在于 SocketsHttpHandler 提供了更多的控制性，如连接超时时间，以及更新 DNS 解析时间和更多的 SSL 控制。由于 HttpClientHandler 是 .NET Framework 4.5 时就提供的类型，比 .NET Core 2.1 提供的 SocketsHttpHandler 类型可使用的 API 数量要少。如果你需要对网络有更多的控制，那还请采用 SocketsHttpHandler 类型

```csharp
var socketsHttpHandler = new SocketsHttpHandler()
{
    // 这个可以设置连接的超时时间
    ConnectTimeout = TimeSpan.FromSeconds(10),
    //PooledConnectionIdleTimeout = TimeSpan.FromSeconds(100),
    // HttpClient only resolves DNS entries when a connection is created. It does not track any time to live (TTL) durations specified by the DNS server. If DNS entries change regularly, which can happen in some container scenarios, the client won't respect those updates. To solve this issue, you can limit the lifetime of the connection by setting the SocketsHttpHandler.PooledConnectionLifetime property, so that DNS lookup is required when the connection is replaced.
    PooledConnectionLifetime = TimeSpan.FromSeconds(1000),
    SslOptions = new SslClientAuthenticationOptions()
    {
        RemoteCertificateValidationCallback = (sender, certificate, chain, errors) => true, // HttpClientHandler.DangerousAcceptAnyServerCertificateValidator 忽略证书错误
    },
};

var httpClient = new HttpClient(socketsHttpHandler);
```

采用 SocketsHttpHandler 能够做到更多的平台独立，从而让网络行为在各个平台统一

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

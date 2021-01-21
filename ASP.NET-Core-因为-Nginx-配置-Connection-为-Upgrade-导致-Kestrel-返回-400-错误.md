
# ASP.NET Core 因为 Nginx 配置 Connection 为 Upgrade 导致 Kestrel 返回 400 错误

我今天遇到了一个坑，我的服务器在经过了 Nginx 之后，发送的 POST 请求，如果请求里面有 Body 内容，那么 Kestrel 将会返回 400 错误，同时也不会经过任何的中间件

<!--more-->


<!-- 发布 -->

在 HTTP 的标准里面，在 HTTP 协议提供了一种特殊的机制，这一机制允许将一个已建立的连接升级成新的、不相容的协议。由客户端发起给服务端询问可以服务器端选择是否要升级到新协议，这个机制可以做到如客户端使用HTTP/1.1去连接服务器端，询问服务器端是否能升级到HTTP2甚至是WebSockets协议。而这个机制的做法如 [mozilla 协议升级机制](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Protocol_upgrade_mechanism) 文档所说，在客户端请求的时候将会添加两个额外的 Header 内容：

Connection: Upgrade    设置 Connection 头的值为 "Upgrade" 来指示这是一个升级请求

Upgrade: protocols     Upgrade 头指定一项或多项协议名，按优先级排序，以逗号分隔

一个典型的包含升级请求的例子差不多是这样的：

```
GET /foo HTTP/1.1
Host: www.example.com
Connection: upgrade
Upgrade: example/1, foo/2
```

而在我这边其实是为了让 Nginx 支持 WebSockets 协议，如 [nginx 反向代理websocket – A Blog](https://blog.sdlsj.net/archives/nginx/nginx-reverse-proxy-websocket/ ) 所说方法，配置如下

```
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade"; 
```

可以看到这里的锅就是，无论是否有配置 Upgrade 的内容，都给 Connection 加上了 upgrade 的内容

咱可以来写一个简单的 demo 程序，尝试在 ASP.NET Core 应用发送一个 POST 请求，这个请求里面包含了这两个 Header 信息，如下面代码

```csharp
            var httpClient = new HttpClient();

            httpClient.DefaultRequestHeaders.Clear();
            httpClient.DefaultRequestHeaders.Add("Upgrade", "");
            httpClient.DefaultRequestHeaders.Add("Connection", "Upgrade");
```

这个 demo 代码放在[gitee](https://gitee.com/lindexi/lindexi_gd/tree/314e0946/HekecicalLechurlaiberlefofe)欢迎大家访问

这个 demo 里面，可以看到只有使用 UseExceptionHandler 的如下重载方法才会进入，而其他的重载方法进入失败

```csharp
            app.UseExceptionHandler(builder =>
            {
                // 这是会进来的
            });
```

而这个问题，官方也有收到反馈，请看 ["Connection: upgrade" causes 400 error that never reaches application code. Triggered by common nginx config. · Issue #17081 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/issues/17081 )

虽然这样做是不符合规范的，但是用的人多了，也就约定俗成了。而 Kestrel 比较严格的遵守标准却在此时挖了一个坑。最近有一个 PR 是允许忽略掉加上 upgrade 在 POST 带上 Body 的逻辑合入到 dotnet core 2.1 和 dotnet core 3.1 和 dotnet 5.0 版本，也许在你看到这个博客的时候，咱的应用其实能做到默认支持的

而其实在官方文档里面也给出了推荐的 Nginx 的配置，如下

```
server {
    listen        80;
    server_name   example.com *.example.com;
    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

可以看到在官方的配置里面给 Connection 配置的是 keep-alive 哈

特别感谢 [lsj](https://blog.sdlsj.net) 的协助，以及运维小伟大佬的方法

而我现在还有一个问题，我可以如何在遇到这样的问题的时候，通过我的应用的日志了解到

更多请看

["Connection: upgrade" causes 400 error that never reaches application code. Triggered by common nginx config. · Issue #17081 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/issues/17081 )

[[5.0] Allow/ignore upgrades with bodies by Tratcher · Pull Request #28896 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/pull/28896 )

[[3.1] Allow/ignore upgrades with bodies by Tratcher · Pull Request #28907 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/pull/28907 )

[[2.1] Allow/ignore upgrades with bodies by Tratcher · Pull Request #28908 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/pull/28908 )


[nginx 反向代理websocket – A Blog](https://blog.sdlsj.net/archives/nginx/nginx-reverse-proxy-websocket/ )

[Configure ASP.NET Core to work with proxy servers and load balancers](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-5.0 )

[Host ASP.NET Core on Linux with Nginx](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-5.0 )

[协议升级机制 - HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Protocol_upgrade_mechanism )

[Connection - HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Connection )

[Kestrel returns 400 before reaching any of my code · Issue #4726 · dotnet/aspnetcore](https://github.com/dotnet/aspnetcore/issues/4726 )

[How to log automatic 400 responses on model validation errors · Issue #12157 · dotnet/AspNetCore.Docs](https://github.com/dotnet/AspNetCore.Docs/issues/12157 )

[Configure options for the ASP.NET Core Kestrel web server](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/options?view=aspnetcore-5.0&WT.mc_id=DX-MVP-5003606 )

[Handle errors in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-5.0&WT.mc_id=DX-MVP-5003606 )

[c# - How to auto log every request in .NET Core WebAPI? - Stack Overflow](https://stackoverflow.com/questions/45479094/how-to-auto-log-every-request-in-net-core-webapi?WT.mc_id=DX-MVP-5003606 )









<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
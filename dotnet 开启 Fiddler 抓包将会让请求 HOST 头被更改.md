# dotnet 开启 Fiddler 抓包将会让请求 HOST 头被更改

我在写域名备份功能，想要修改请求的 IP 地址，同时又将原有的请求域名带上。实现方法是修改请求的地址，在 HttpRequestMessage 的 Header 上添加 HOST 记录，记录的值就是原有的域名。然而在开启 Fiddler 之后，将会发现实际发出的请求的 HOST 是实际请求的地址

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

预计这个是 Fiddler 的已知问题

我的实现方法是给 HttpRequestMessage 的 Headers 设置上 Host 的内容，接着修改 RequestUri 为某个 IP 地址，从而实现域名备份功能。域名备份是我为了解决国内诡异的网络环境的问题而引入的技术方案，域名备份的实现方法是运维将后台在多个地方提供访问入口，可选的是将后台在多个城市部署多套，或者只部署一套后台但是在其他城市提供可访问入口，如内部代理等方式。客户端拿到后台的多个访问域名，如果首选域名访问不通，则尝试使用备份的域名进行访问。甚至有时期望能绕过 DNS 污染，直接访问已知的后台 IP 访问后台

此时需要保持行为一致，让后台可以拿到的请求 Host 保持和原来一样。这就是本文所记录的问题的遇到的原因。我的代码如下：

```csharp
        var httpRequestMessage = new HttpRequestMessage();
        httpRequestMessage.Headers.Host = host;
        httpRequestMessage.Method = HttpMethod.Post;
        httpRequestMessage.RequestUri = new Uri(actualHost);

        using var httpClient = new HttpClient();
        var httpResponseMessage = await httpClient.SendAsync(httpRequestMessage);
```

为了方便测试，我加上了测试使用的后台，测试后台的框架代码如下：

```csharp
class TestHost : IDisposable
{
    public TestHost(string host, WebApplication app)
    {
        Host = host;
        App = app;
    }

    public string Host { get; }
    public WebApplication App { get; }

    public void Dispose()
    {
        App.DisposeAsync();
    }
}

static class TestHostBuilder
{
    public static TestHost GetTestHost(Action<WebApplication> configure)
    {
        var port = GetAvailablePort();
        var builder = WebApplication.CreateBuilder();
        var host = $"http://127.0.0.1:{port}/";
        builder.WebHost.UseUrls(host);
        builder.Services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();

        var app = builder.Build();

        configure(app);

        _ = app.RunAsync();

        return new TestHost(host, app);
    }

    public static int GetAvailablePort()
    {
        return GetAvailablePort(IPAddress.Loopback);
    }

    /// <summary>
    /// 获取一个可用端口
    /// </summary>
    /// <param name="ip"></param>
    /// <returns></returns>
    public static int GetAvailablePort(IPAddress ip)
    {
        using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
        socket.Bind(new IPEndPoint(ip, 0));
        socket.Listen(1);
        var ipEndPoint = (IPEndPoint) socket.LocalEndPoint!;
        var port = ipEndPoint.Port;
        return port;
    }
}
```

测试后台是通过在本机开启一个随机的端口然后提供访问的方式，具体实现如下

```csharp
        var host = "unknownaddressxxxxxxxxxxxasdxx.xxxxxx";

        using var testHost = TestHostBuilder.GetTestHost(app =>
        {
            app.MapPost("/", () =>
            {
                var httpContextAccessor = app.Services.GetRequiredService<IHttpContextAccessor>();

                // 测试可以拿到域名
                if (httpContextAccessor.HttpContext!.Request.Host.Host == host)
                {
                    return "Return";
                }

                return null;
            });
        });
```

以上使用 IHttpContextAccessor 可以拿到 HttpContext 对象，从而获取到请求的 Host 内容。上面测试代码写了一个叫 `unknownaddressxxxxxxxxxxxasdxx.xxxxxx` 的不存在的域名，期望能在测试的后台里面收到的请求使用此域名

修改调用的代码如下

```csharp
        var httpRequestMessage = new HttpRequestMessage();
        httpRequestMessage.Headers.Host = host;
        httpRequestMessage.Method = HttpMethod.Post;
        httpRequestMessage.RequestUri = new Uri(testHost.Host);

        using var httpClient = new HttpClient();
        var httpResponseMessage = await httpClient.SendAsync(httpRequestMessage);
        var result = await httpResponseMessage.Content.ReadAsStringAsync();
        Console.WriteLine(result);
```

在开启了 Fiddler 抓包工具之后，抓到的请求如下

```
POST http://127.0.0.1:50662/ HTTP/1.1
Host: 127.0.0.1:50662
Content-Length: 0
```

测试后台收到的 Host 也是 `127.0.0.1:50662` 而不是期望的测试域名

关闭 Fiddler 抓包工具之后，在测试后台可以收到期望的测试域名

因此抓包工具 Fiddler 将会篡改请求的 Host 信息为请求的实际地址的域名。我不知道如何配置 Fiddler 禁用此行为，还请知道如何配置的大佬教教我

本文的测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/5066e8e0a494bfe571210bb3dfa748c27209e986/HiwelairchawCekaywebeale) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5066e8e0a494bfe571210bb3dfa748c27209e986/HiwelairchawCekaywebeale) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5066e8e0a494bfe571210bb3dfa748c27209e986
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5066e8e0a494bfe571210bb3dfa748c27209e986
```

获取代码之后，进入 HiwelairchawCekaywebeale 文件夹
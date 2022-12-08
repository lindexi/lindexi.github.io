# dotnet core 不自动从 https 到 http 的 302 重定向

本文记录一个已知问题，或者准确来说是设计如此的行为，在 dotnet core 下，无论是 dotnet core 3.1 还是 dotnet 5 或 dotnet 6 或 dotnet 7 等，如果访问的 https 链接返回 302 状态码，且跳转的链接是一个 http 链接，那将不会自动跳转

<!--more-->
<!-- CreateTime:2022/12/7 19:36:33 -->
<!-- 发布 -->
<!-- 博客 -->

默认情况下，咱可以通过设置 HttpClient 的 HttpClientHandler 从而设置 AllowAutoRedirect 属性，让 HttpClient 自动执行 302 跳转，且可以加上 MaxAutomaticRedirections 设置允许最大的跳转次数

```csharp
private static HttpClient _httpClient = new HttpClient
(
    new HttpClientHandler 
    { 
        AllowAutoRedirect = true, 
        MaxAutomaticRedirections = 2 
    }
);
```

对应的，在 dotnet 6 或更高的版本，可以使用 SocketsHttpHandler 代替 HttpClientHandler 类型，代替之后的代码其实也差不多，请看以下代码，更多请看 [dotnet 6 HttpClientHandler 和 SocketsHttpHandler 有什么差别](https://blog.lindexi.com/post/dotnet-6-HttpClientHandler-%E5%92%8C-SocketsHttpHandler-%E6%9C%89%E4%BB%80%E4%B9%88%E5%B7%AE%E5%88%AB.html )

```csharp
            HttpMessageHandler handler = new SocketsHttpHandler()
            {
                AllowAutoRedirect = true,
                MaxAutomaticRedirections = 10,
            };
            using var httpClient = new HttpClient(handler);
```

这在大部分情况下都能正常工作，但是如果所访问的链接是一个 https 链接，且此链接返回 302 跳转到一个 http 链接上，那使用 HttpClient 将不会自动跳转，而是返回 302 的状态码，且在 Header 的 Location 上写明了后台返回的 http 链接

这是 dotnet core 的设计如此，可以通过本文的参考看到大佬们的讨论

由于从 https 跳转到 http 在大部分时候来说，都是十分诡异的行为。默认不要让 HttpClient 帮助自动跳转也是十分符合预期的行为

如果自己明确知道没有问题，那就自己加上跳转的代码吧

如以下的例子代码，先判断 StatusCode 是 Redirect 然后拿 Headers.Location 重新访问

```csharp
            var httpResponseMessage = await httpClient.GetAsync(url);
            var resultResponseMessage = httpResponseMessage;

            if (httpResponseMessage.StatusCode == HttpStatusCode.Redirect)
            {
                var location = httpResponseMessage.Headers.Location;
                if (location is not null)
                {
                    var newResponseMessage = await httpClient.GetAsync(location);
                    resultResponseMessage = newResponseMessage;
                }
                else
                {
                    // 理论上不能为空吧，抛个异常还是返回就看你业务
                }
            }
```

默认行为禁止 https->http 的跳转，是一个很合理的设计。如果明确知道后台想要如此行为，最好先去将后台的伙伴打一顿，如果打不过，再考虑按照以上代码的方式更改

参考：

[HttpClient does not follow 302 redirects · Issue #23801 · dotnet/runtime](https://github.com/dotnet/runtime/issues/23801 )

[HttpClient follow 302 redirects with .NET Core Brian Pedersen's Sitecore and .NET Blog](https://briancaos.wordpress.com/2021/09/06/httpclient-follow-302-redirects-with-net-core/ )

[Problem with HttpClient after updating my project from net461 to netcoreapp2.0 · Issue #23697 · dotnet/runtime](https://github.com/dotnet/runtime/issues/23697 )

[Response status code does not indicate success: 302 (Moved Temporarily). · Issue #894 · TelegramBots/Telegram.Bot](https://github.com/TelegramBots/Telegram.Bot/issues/894 )

[Change System.Net.Http to throw exception for HTTPS -> HTTP redirects · Issue #23813 · dotnet/runtime](https://github.com/dotnet/runtime/issues/23813 )

[Log when an insecure Https -> Http redirect is blocked by rmkerr · Pull Request #27077 · dotnet/corefx](https://github.com/dotnet/corefx/pull/27077/files )
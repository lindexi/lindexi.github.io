本文记录一个 dotnet 的设计问题，默认创建出来的 JsonContent 对象的 Headers 里，是没有 Content-Length 信息的

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

如下面代码创建一个 JsonContent 对象

```csharp
using System.Net.Http.Json;

var foo = new Foo();

var jsonContent = JsonContent.Create(foo);

class Foo
{
    public int Value { set; get; }
}
```

此时如果遍历 JsonContent 的 Headers 属性，将只可以拿到 Content-Type 信息，没有 Content-Length 信息

在现代的绝大部分服务端，都是支持 Content 不带 Content-Length 信息的，这在大部分后台上都能正常符合预期工作

即使用大概如下代码的 JsonContent 发送出去的请求，在请求里面也是不带 Content-Length 信息的

```csharp
    var foo = new Foo();
    var jsonContent = JsonContent.Create(foo);

    var httpClient = new HttpClient();
    await httpClient.PostAsync("https://blog.lindexi.com", jsonContent);
```

那如何可以让 JsonContent 带上 Content-Length 信息？只需调用 LoadIntoBufferAsync 方法，如以下代码

```csharp
var jsonContent = JsonContent.Create(foo);

await jsonContent.LoadIntoBufferAsync();
```

调用完成 LoadIntoBufferAsync 方法，即可在 Headers 里面看到 Content-Length 信息，且使用如下代码发送请求也是带上 Content-Length 信息的

```csharp
    await jsonContent.LoadIntoBufferAsync();

    var httpClient = new HttpClient();
    await httpClient.PostAsync("https://blog.lindexi.com", jsonContent);
```

我查看请求的信息是通过自己创建一个简单的 ASP.NET Core 程序，代码大概如下

```csharp
var builder = WebApplication.CreateSlimBuilder(args);

var app = builder.Build();

app.MapPost("/", async context =>
{
    await Task.CompletedTask;
    var headers = context.Request.Headers;
});

app.Run();
```

通过断点在 `var headers = context.Request.Headers;` 即可了解客户端请求发送过来的请求头信息

以及将此请求尝试发送到其他服务器上，通过抓包确定了具体的行为

这在 dotnet 里面认为设计如此，且认为如果没有足够多的报告说缺少 Content-Length 信息会让后台不工作，则依然保持此行为

讨论内容请看：

[.NET 6: JsonContent.Create(obj) should set Content-Length HTTP request header · Issue #70793 · dotnet/runtime](https://github.com/dotnet/runtime/issues/70793 )

[Content-Length not appended when using JsonContent · Issue #82984 · dotnet/runtime](https://github.com/dotnet/runtime/issues/82984 )

[Provide better json support for servers don't support chunked request body (re-open) · Issue #55583 · dotnet/runtime](https://github.com/dotnet/runtime/issues/55583 )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1b312eb1bfb867e56c5bbc61df720819fe1e15fc/Workbench/CaiballkaylecaWairlaroweneno) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1b312eb1bfb867e56c5bbc61df720819fe1e15fc/Workbench/CaiballkaylecaWairlaroweneno) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1b312eb1bfb867e56c5bbc61df720819fe1e15fc
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1b312eb1bfb867e56c5bbc61df720819fe1e15fc
```

获取代码之后，进入 Workbench/CaiballkaylecaWairlaroweneno 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

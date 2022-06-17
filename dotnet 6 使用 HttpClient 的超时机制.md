# dotnet 6 使用 HttpClient 的超时机制

在 dotnet 6 里，推荐的网络通讯是使用 HttpClient 类型，在国内诡异的网络环境下，有很多弱网环境需要考虑，其中很重要一点就是网络超时。本文将来告诉大家如何合理使用 HttpClient 的超时机制

<!--more-->
<!-- 发布 -->

在 HttpClient 里面有一个 Timeout 属性，这个属性的含义是整个网络活动过程中的超时时间，这个定义是有一定的坑的。例如我对一个 API 数据接口进行访问，只是 POST 一段很短的数据，此时设置超时时间是 100 秒，默认超时时间是完全合理的。但是如果我是进行一个大文件上传，文件上传的时间很长，那此时采用超时时间是 100 秒显然是不合理的，在 100 秒内如果文件还没上传完成，也就是网络活动还没完成，将会触发超时异常

这是比较非预期的逻辑，大部分情况下，在国内的弱网环境下，可能在上传过程中，网络几乎被断开，网络几乎被断开等同于上传速度非常慢。整个文件上传过程可以分为两个阶段，第一个阶段和服务器建立连接的过程，这个过程如果采用 HttpClient 的 Timeout 属性作为超时时间，那是比较合理的。第二个阶段是上传数据过程，这个过程的时间完全和上传的数据量相关。显然，如果将第二个阶段也计算入超时时间范围内，是不符合预期的

在使用 HttpClient 时，对于大部分的网络请求，非上传文件的情况下，这个超时的时间都是符合预期的逻辑。而对于上传文件的情况，这是不符合预期的逻辑，更多的时候，需要的是，如果上传的速度慢到一定程度的时候，报告超时给到业务端。例如上传的速度很长时间就几乎为零，那就应该报给上层业务端

但文件上传过程如上文，可以分为两个阶段。可以通过更底层控制的方法设置 HttpClient 的和服务器连接的超时时间，代码如下

```csharp
    var socketsHttpHandler = new SocketsHttpHandler()
    {
        ConnectTimeout = TimeSpan.FromSeconds(20),
    };
    var httpClient = new HttpClient(socketsHttpHandler)
    {
        Timeout = TimeSpan.FromSeconds(100)
    };
```

在 HttpClient 里面传入 SocketsHttpHandler 对象，可以在 SocketsHttpHandler 对象进行更底层的控制，从而实现控制连接超时时间。在 dotnet 6 下，默认的 HttpClient 底层就是调用 SocketsHttpHandler 对象，因此以上代码对 HttpClient 底层行为没有任何变更

有些伙伴在遇到此问题的时候，在网上搜到了一些上古的解决方案，那就是采用 HttpWebRequest 的方式。然而坏消息是在 dotnet 6 下，由于 HttpWebRequest 的底层就是采用 HttpClient 实现，因此 HttpWebRequest 是解决不了此问题的。详细请看 [dotnet 6 使用 HttpWebRequest 进行 POST 文件将占用大量内存](https://blog.lindexi.com/post/dotnet-6-%E4%BD%BF%E7%94%A8-HttpWebRequest-%E8%BF%9B%E8%A1%8C-POST-%E6%96%87%E4%BB%B6%E5%B0%86%E5%8D%A0%E7%94%A8%E5%A4%A7%E9%87%8F%E5%86%85%E5%AD%98.html )

一个实现机制也如官方所说，如果要对上传逻辑有足够的控制，那请用好 PostAsync 最后一个参数，也就是说一个好的方式是将 HttpClient 的上传大量数据分为两个超时阶段。第一个阶段是连接阶段，通过 SocketsHttpHandler 的 ConnectTimeout 控制，第二个阶段是通过 PostAsync 的取消参数控制

实现方法是先将 HttpClient 的 Timeout 设置为一个足够长的时间，甚至可以使用 `Timeout.InfiniteTimeSpan` 属性设置为无穷时间超时，然后靠取消参数控制超时

```csharp
    var socketsHttpHandler = new SocketsHttpHandler()
    {
        ConnectTimeout = TimeSpan.FromSeconds(20),
    };
    var httpClient = new HttpClient(socketsHttpHandler)
    {
        Timeout = Timeout.InfiniteTimeSpan
    };
```

接下来再定义一个 UploadHttpContent 类型，继承 HttpContent 类型，用来做实际上的上传速度控制逻辑

```csharp
class UploadHttpContent : HttpContent
{

}
```

需要传入实际上文件上传数据的 HttpContent 内容，和设置的超时时间

```csharp
    public UploadHttpContent(HttpContent content, CancellationTokenSource tokenSource, TimeSpan? timeout = null)
    {
        _content = content;
        _tokenSource = tokenSource;
        _stream = content.ReadAsStream();
        _timeout = timeout ?? TimeSpan.FromSeconds(100);
    }

    private TimeSpan _timeout;

    private readonly HttpContent _content;
    private Stream _stream;
    private CancellationTokenSource _tokenSource;
```

这里的超时时间定义不是上传的总时间，而是上传过程中网络断开的时间。这里的网络断开是等同于网络速度足够慢，例如定义为经过了 100 秒还上传不了 1 MB 的数据，那就上报超时

先忽略 UploadHttpContent 的实现逻辑，先看一下使用的方法

先获取到一个上传的数据，以下采用一个测试用的 Stream 代替

```csharp
var streamContent = new StreamContent(new FakeStream(1024_0000_0000));
```

这里的 FakeStream 可以产生如参数传给他的数据量，可以看到这是一个比较大的数据

再定义取消的参数

```csharp
var cancellationTokenSource = new CancellationTokenSource();
```

接着创建 UploadHttpContent 对象

```csharp
var uploadHttpContent = new UploadHttpContent(streamContent, cancellationTokenSource);
```

将 UploadHttpContent 作为上传的参数，代码如下

```csharp
var result = await httpClient.PostAsync("http://127.0.0.1:12367/Upload", uploadHttpContent, cancellationTokenSource.Token);
```

在 UploadHttpContent 里面，通过重写 SerializeToStreamAsync 方法，可以在每次上传缓存读取时进入方法。每次进入方法可以记录间隔时间，从而实现通过间隔时间判断上传超时

```csharp
class UploadHttpContent : HttpContent
{
    // 忽略其他逻辑
    protected override async Task SerializeToStreamAsync(Stream stream, TransportContext? context)
    {
        var buffer = ArrayPool<byte>.Shared.Rent(1024 * 1024);
        int count;

        StartDog();

        while ((count = _stream.Read(buffer, 0, buffer.Length)) > 0)
        {
            // 这里存在一个问题是如果先读取完成了缓存，然后发送慢了，依然会炸掉
            _stopwatch.Restart();

            await stream.WriteAsync(new ReadOnlyMemory<byte>(buffer, 0, count), _tokenSource.Token);
        }
    }

    private readonly Stopwatch _stopwatch = new Stopwatch();
}
```

在进入 SerializeToStreamAsync 方法时，也就是开始发起请求时，将开启 StartDog 方法。进入 SerializeToStreamAsync 方法是不需要等待和服务器连接开始就调用的，因为在底层调用 SerializeToStreamAsync 方法是先将数据读取到缓存里面，在建立连接完成之后，将从缓存里面发送数据给服务器。这样的设计的原因是为了提升性能，如果是在连接完成之后再进行读取 SerializeToStreamAsync 方法，那将会导致连接完成之后需要等待一下才能从业务端读取到数据

在进入第一次读取调用 StartDog 将进入一个循环逻辑，在这里面判断 `_stopwatch` 字段，从而了解到调用的频率。此读取的频率约等于网络上传的速率，但是需要了解的是输入参数的 stream 是本地的缓存。在本地缓慢满的时候，调用 WriteAsync 方法将不会返回

```csharp
    private async void StartDog()
    {
        while (!_isFinished)
        {
            await Task.Delay(_timeout / 2);
            if (_isFinished)
            {
                return;
            }

            if (_stopwatch.Elapsed > _timeout)
            {
                _tokenSource.Cancel();
                return;
            }
        }
    }

    private bool _isFinished;

    public void SetIsFinished() => _isFinished = true;
```

在 StartDog 里面大概等待时间间隔是 `_timeout / 2` 的值，在这个范围内判断是否有 `_stopwatch` 距离上次开启的时间超过 `_timeout` 的值，如果超过了，那就证明网络速度足够慢。这里的等待间隔选用 `_timeout / 2` 的值，最差等待超时时间将会是实际超时的 1.5 倍时间，如果关心超时时间，那请将这个间隔设置比较小

以上代码的 SetIsFinished 是设计给上传完全完成之后调用的，如果不调用问题也不大，因此最后也会判断超时而返回，只是这个最后判断设置的逻辑是没有实际使用的

```csharp
var uploadHttpContent = new UploadHttpContent(streamContent, cancellationTokenSource);

var result = await httpClient.PostAsync("http://127.0.0.1:12367/Upload", uploadHttpContent, cancellationTokenSource.Token);
uploadHttpContent.SetIsFinished(); // 设置完成
```

如果去掉以上的 `SetIsFinished` 方法，修改为在 SerializeToStreamAsync 方法调用结束的时候设置 `_isFinished` 的值，那存在一个小问题，那就是进入 SerializeToStreamAsync 方法的循环最后一次是将数据写入到缓存里面，假设网络速度在发送最后的缓存数据是比较慢的，那无疑没有后续的判断逻辑可以告诉超时时间。为了解决此问题，才有了 SetIsFinished 方法，在实际上的 Post 完成之后，再进行设置。当然了此时不设置问题也不大，只是多了一次无效的超时调用

接下来写一点测试代码，在服务器端设置了上传将会是一个缓慢读取的方式，如下面代码

```csharp
using System.Buffers;

WebApplicationBuilder builder = WebApplication.CreateBuilder(args);
builder.WebHost.UseUrls("http://*:12367");
builder.WebHost.UseKestrel(options =>
{
    options.Limits.MaxRequestBodySize = 1024_0000_0000_0000_000;
});
var app = builder.Build();

app.MapPost("/Upload", async context =>
{
    var length = 1024 * 1024 * 100;
    var buffer = ArrayPool<byte>.Shared.Rent(length);

    int count;
    while ((count = await context.Request.Body.ReadAsync(buffer, 0, length)) > 0)
    {
        await Task.Delay(1000);
    }

    ArrayPool<byte>.Shared.Return(buffer);

    context.Response.StatusCode = StatusCodes.Status200OK;
    await context.Response.WriteAsync("Hello World!");
});

app.Run();
```

以上的服务器端的接收客户端上传的速度是可以接受的，每次读取都等待一秒的时间，这比设置的超时时间短，因此调用 Upload 上传是不会超时的

再写另一个服务器端的方法，这个方法接收数据会更加慢，比设置的超时时间慢

```csharp
app.MapPost("/UploadTimeout", async context =>
{
    var length = 1024 * 1024 * 100;
    var buffer = ArrayPool<byte>.Shared.Rent(length);

    int count;
    int n = 0;
    while ((count = await context.Request.Body.ReadAsync(buffer, 0, length)) > 0)
    {
        await Task.Delay(1000);
        n++;
        if (n == 10)
        {
            await Task.Delay(TimeSpan.FromHours(10));
        }
    }

    ArrayPool<byte>.Shared.Return(buffer);

    context.Response.StatusCode = StatusCodes.Status200OK;
    await context.Response.WriteAsync("Hello World!");
});
```

此时的客户端上传将会被提示超时

以上逻辑即可实现让客户端上传大量数据时，通过上传的速度设置超时，可以比较好解决国内的弱网环境

以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/3015fafa0a38e1eb98b0b7eed117f46911253ea4/NekejawcharlereJibabearcel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/3015fafa0a38e1eb98b0b7eed117f46911253ea4/NekejawcharlereJibabearcel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 3015fafa0a38e1eb98b0b7eed117f46911253ea4
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NekejawcharlereJibabearcel 文件夹

但是 HttpClient 的 Timeout 属性对于下载过程是不做限制的，也就是在请求上之后进行下载的过程，如果下载时间超过了 Timeout 设置的时间，依然能继续下载

测试下载超时的影响的代码，在服务端添加如下代码，用来提供一个非常大的数据给客户端下载

```csharp
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);
builder.WebHost.UseUrls("http://*:12367");
builder.WebHost.UseKestrel(options =>
{
    options.Limits.MaxRequestBodySize = 1024_0000_0000_0000_000;
});
var app = builder.Build();

app.MapGet("/Download", async context =>
{
    var length = 1024 * 1024 * 100;
    var buffer = ArrayPool<byte>.Shared.Rent(length);

    for (int i = 0; i < 1000000; i++)
    {
        await context.Response.Body.WriteAsync(new ReadOnlyMemory<byte>(buffer));

        if (i < 10)
        {
            await Task.Delay(TimeSpan.FromSeconds(1));
        }
        else
        {
            await Task.Delay(TimeSpan.FromMinutes(1));
        }
    }
});
app.Run();
```

客户端设置超时 10 秒，然后进行下载，以下代码一定是 10 秒下载不完成的

```csharp
async Task Download()
{
    var httpClient = new HttpClient()
    {
        Timeout = TimeSpan.FromSeconds(10)
    };

   var stream = await httpClient.GetStreamAsync("http://127.0.0.1:12367/Download");

   var count = 0;
   var buffer = ArrayPool<byte>.Shared.Rent(1024 * 1024);

   while ((count = stream.Read(buffer.AsSpan())) > 0)
   {
       Console.WriteLine($"{count}");
   }
}
```

可以看到下载超过了 10 秒还能继续下载，证明了 Timeout 属性对下载是无效的

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  


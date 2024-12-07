# ASP.NET Core 简单给 Phi 模型封装一个服务

本文将和大家介绍如何使用 ASP.NET Core 简单给 Phi 模型封装成一个服务，可以让其他设备通过 http 请求方式调用到模型计算能力

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

我师弟跑路了，留给我一台特别好的设备，一台带 NVIDIA GeForce RTX 3090 Ti 显卡的设备。我在这台设备上可以轻松通过 DirectML 跑起来 Phi 模型。既然已经跑起来模型了，我就想着能否用这个模型给更多的应用赋能。其中我想到的第一步就是搭建一个 Http 服务，让其他设备可以调用到这台好设备，从而在这台好设备上跑模型，得到的结果通过 http 返回给到其他设备

这样一来，其他设备也就能享受到 Phi 模型带来的智能化

在 ASP.NET Core 框架的帮助下，给 Phi 模型调用封装一个服务是非常简单的事情，我甚至在一个 Program.cs 文件里面不到 200 行代码就完成了

首先是基于 [dotnet 基于 DirectML 控制台运行 Phi-3 模型](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-DirectML-%E6%8E%A7%E5%88%B6%E5%8F%B0%E8%BF%90%E8%A1%8C-Phi-3-%E6%A8%A1%E5%9E%8B.html ) <!-- [dotnet 基于 DirectML 控制台运行 Phi-3 模型 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18245125 ) --> 博客提供的方法用 DirectML 跑起来 Phi-3 模型。如何用 DirectML 跑起来 Phi-3 模型部分本文就一笔带过，感兴趣的伙伴还请阅读我之前的博客。放心，全部的代码都可以在本文末尾找到下载的方法

准备的代码如下

```csharp
var folder = @"C:\lindexi\Phi3\directml-int4-awq-block-128\";
if (!Directory.Exists(folder))
{
    folder = Path.GetFullPath(".");
}

Model model = new Model(folder);
```

还请大家将文件夹路径更换为自己的模型文件夹

尽管 NVIDIA GeForce RTX 3090 Ti 显卡已经很强大了，但这台设备日常还有其他活。为了不被点爆。我这里添加了一个 SemaphoreSlim 用来控制并发量，代码如下

```csharp
var semaphoreSlim = new SemaphoreSlim(initialCount: 1, maxCount: 1);
```

每次进入请求的时候，都会等待一下 SemaphoreSlim 从而控制进入到模型计算里，每次最多只有一次请求。其他请求就依次进行排队。为什么能依次呢？因为这是利用了 SemaphoreSlim 带来的额外功能，详细请看 [C# dotnet 的锁 SemaphoreSlim 和队列](https://blog.lindexi.com/post/C-dotnet-%E7%9A%84%E9%94%81-SemaphoreSlim-%E5%92%8C%E9%98%9F%E5%88%97.html )

再以下就是通用的初始化 ASP.NET Core 主机的代码

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.WebHost.UseUrls("http://0.0.0.0:5017");
builder.Services.AddLogging(loggingBuilder => loggingBuilder.AddSimpleConsole());

var app = builder.Build();
```

我约定了其他设备的请求采用的是 HTTP 的 POST 请求方式，为了接受和约定请求，我就定义了名为 ChatRequest 的类型。这个类型只有 Prompt 提示词一个参数，因为这正是模型所需的必备参数

```csharp
record ChatRequest(string Prompt)
{
}
```

完成请求类的定义之后，接下来即可声明 POST 的路径映射了，代码如下

```csharp
app.MapPost("/Chat", async (ChatRequest request, HttpContext context, [FromServices] ILogger<ChatSessionLogInfo> logger) =>
{
     ... // 忽略其他代码
});
app.Run();
```

以上代码传入的委托里面，只有 ChatRequest 是从其他设备发送过来的，其他两个参数都是框架提供的

先设置 http 响应，包括设置状态码和设置开始，代码如下

```csharp
app.MapPost("/Chat", async (ChatRequest request, HttpContext context, [FromServices] ILogger<ChatSessionLogInfo> logger) =>
{
    var response = context.Response;
    response.StatusCode = (int) HttpStatusCode.OK;
    await response.StartAsync();

     ... // 忽略其他代码
});
```

设置状态码一定要在 StartAsync 之前，调用 StartAsync 时，客户端将会收到 HTTP 的响应头了。在 StartAsync 和 CompleteAsync 之间，就可以慢慢发送 Body 过去

进入 StartAsync 之后，再等待信号量。如此可以先让客户端收到 HTTP 的头，不会让客户端等得太无聊

<!-- 此时就不会让客户端请求的时候，发现 HTTP 超时了，默认的 HttpClient 的设计上是只要收到头了，就认为请求已经开始了，其超时的属性控制就不会再生效 -->

```csharp
app.MapPost("/Chat", async (ChatRequest request, HttpContext context, [FromServices] ILogger<ChatSessionLogInfo> logger) =>
{
    var response = context.Response;
    response.StatusCode = (int) HttpStatusCode.OK;
    await response.StartAsync();
    await semaphoreSlim.WaitAsync();

    try
    {
         ... // 忽略其他代码
    }
    finally
    {
        semaphoreSlim.Release();
        await response.CompleteAsync();
    }
});
```

完成基础框架逻辑之后，咱就可以开始让模型处理其他设备发送过来的提示词信息，核心代码如下

```csharp
        var prompt = request.Prompt;

        logger.LogInformation($"Session={sessionName};TraceId={traceId}\r\nPrompt={request.Prompt}");

        var generatorParams = new GeneratorParams(model);

        using var tokenizer = new Tokenizer(model);
        var sequences = tokenizer.Encode(prompt);

        generatorParams.SetSearchOption("max_length", 1024);
        generatorParams.SetInputSequences(sequences);
        generatorParams.TryGraphCaptureWithMaxBatchSize(1);

        using var tokenizerStream = tokenizer.CreateStream();
        using var generator = new Generator(model, generatorParams);

        var stringBuilder = new StringBuilder();

        while (!generator.IsDone())
        {
            generator.ComputeLogits();
            generator.GenerateNextToken();

            // 每次只会添加一个 Token 值
            // 需要调用 tokenizerStream 的解码将其转为人类可读的文本
            // 由于不是每一个 Token 都对应一个词，因此需要根据 tokenizerStream 压入进行转换，而不是直接调用 tokenizer.Decode 方法，或者调用 tokenizer.Decode 方法，每次都全部转换

            var text = Decode();

            // 有些时候这个 decodeText 是一个空文本，有些时候是一个单词
            // 空文本的可能原因是需要多个 token 才能组成一个单词
            // 在 tokenizerStream 底层已经处理了这样的情况，会在需要多个 Token 才能组成一个单词的情况下，自动合并，在多个 Token 中间的 Token 都返回空字符串，最后一个 Token 才返回组成的单词
            if (!string.IsNullOrEmpty(text))
            {
                stringBuilder.Append(text);
            }

            await streamWriter.WriteAsync(text);
            await streamWriter.FlushAsync();


            string? Decode()
            {
                // 这里的 tokenSequences 就是在输入的 sequences 后面添加 Token 内容
                ReadOnlySpan<int> tokenSequences = generator.GetSequence(0);
                // 取最后一个进行解码为文本
                var decodeText = tokenizerStream.Decode(tokenSequences[^1]);

                //// 当前全部的文本
                //var allText = tokenizer.Decode(tokenSequences);

                return decodeText;
            }
        }
```

对于服务化来说，以上代码最核心的就是当模型每吐出一个字的时候，就调用一次 `await streamWriter.WriteAsync(text)` 将其发送出去

由于我的业务需求都在一个局域网内，我就无视 `await streamWriter.WriteAsync(text)` 的耗时了。如果大家准备跨网发送，则可以再使用 Channel 进行优化，确保让模型跑得不间断，不会由于网络速度影响而让模型没有全速跑

以上就是使用 ASP.NET Core 简单给 Phi 模型封装一个服务的方法

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/006b67e95fb0e1aa832c5a63c27b78a3fd33ee0a/Bp/JearnugurgelrurkawdoBeacecidem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/006b67e95fb0e1aa832c5a63c27b78a3fd33ee0a/Bp/JearnugurgelrurkawdoBeacecidem) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 006b67e95fb0e1aa832c5a63c27b78a3fd33ee0a
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 006b67e95fb0e1aa832c5a63c27b78a3fd33ee0a
```

获取代码之后，进入 Bp/JearnugurgelrurkawdoBeacecidem 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
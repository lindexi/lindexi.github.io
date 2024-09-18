---
title: dotnet 将本地的 Phi-3 模型与 SemanticKernel 进行对接
description: 在本地完成 Phi-3 模型的部署之后，即可在本地拥有一个小语言模型。本文将告诉大家如何将本地的 Phi-3 模型与 SemanticKernel 进行对接，让 SemanticKernel 使用本地小语言模型提供的能力
tags: dotnet
category: 
---

<!-- CreateTime:2024/06/20 07:05:26 -->

<!-- 发布 -->
<!-- 博客 -->

在我大部分的博客里面，都是使用 AzureAI 和 SemanticKernel 对接，所有的数据都需要发送到远端处理。这在离线的情况下比较不友好，在[上一篇](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-DirectML-%E6%8E%A7%E5%88%B6%E5%8F%B0%E8%BF%90%E8%A1%8C-Phi-3-%E6%A8%A1%E5%9E%8B.html )博客和大家介绍了如何基于 DirectML 控制台运行 Phi-3 模型。本文将在上一篇博客的基础上，告诉大家如何将本地的 Phi-3 模型与 SemanticKernel 进行对接

依然是和[上一篇](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-DirectML-%E6%8E%A7%E5%88%B6%E5%8F%B0%E8%BF%90%E8%A1%8C-Phi-3-%E6%A8%A1%E5%9E%8B.html )博客一样准备好 Phi-3 模型的文件夹，本文这里我放在 `C:\lindexi\Phi3\directml-int4-awq-block-128` 路径下。如果大家下载时拉取不下来 <https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-onnx/tree/main?clone=true> 仓库，可以发送邮件向我要，我将通过网盘分享给大家

准备好模型的下载工作之后，接下来咱将新建一个控制台项目用于演示

编辑控制台的 csproj 项目文件，修改为以下代码用于安装所需的 NuGet 包

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.ML.OnnxRuntimeGenAI.DirectML" Version="0.2.0-rc7" />
    <PackageReference Include="feiyun0112.SemanticKernel.Connectors.OnnxRuntimeGenAI.DirectML" Version="1.0.0" />

    <PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="8.0.0" />
    <PackageReference Include="Microsoft.SemanticKernel" Version="1.13.0" />
  </ItemGroup>
</Project>
```

这里的 `feiyun0112.SemanticKernel.Connectors.OnnxRuntimeGenAI.DirectML` 是可选的，因为最后咱将会自己编写所有对接代码，不需要使用大佬写好的现有组件

先给大家演示使用 `feiyun0112.SemanticKernel.Connectors.OnnxRuntimeGenAI.DirectML` 提供的简单版本。此版本代码大量从 <https://github.com/microsoft/Phi-3CookBook/blob/0a167c4b8045c1b9abb84fc63ca483ae614a88a5/md/07.Labs/Csharp/src/LabsPhi302/Program.cs> 抄的，感谢 Bruno Capuano 大佬

定义或获取本地模型所在的文件夹

```csharp
var modelPath = @"C:\lindexi\Phi3\directml-int4-awq-block-128";
```

创建 SemanticKernel 构建器时调用 `feiyun0112.SemanticKernel.Connectors.OnnxRuntimeGenAI.DirectML` 库提供的 AddOnnxRuntimeGenAIChatCompletion 扩展方法，如以下代码

```csharp
// create kernel
var builder = Kernel.CreateBuilder();
builder.AddOnnxRuntimeGenAIChatCompletion(modelPath);
```

如此即可完成连接逻辑，将本地 Phi-3 模型和 SemanticKernel 进行连接就此完成。接下来的代码就是和原来使用 SemanticKernel 的一样。这一点也可以看到 SemanticKernel 的设计还是很好的，非常方便进行模型的切换

尝试使用 SemanticKernel 做一个简单的问答机

```csharp
var kernel = builder.Build();

// create chat
var chat = kernel.GetRequiredService<IChatCompletionService>();
var history = new ChatHistory();

// run chat
while (true)
{
    Console.Write("Q: ");
    var userQ = Console.ReadLine();
    if (string.IsNullOrEmpty(userQ))
    {
        break;
    }
    history.AddUserMessage(userQ);

    Console.Write($"Phi3: ");
    var response = "";
    var result = chat.GetStreamingChatMessageContentsAsync(history);
    await foreach (var message in result)
    {
        Console.Write(message.Content);
        response += message.Content;
    }
    history.AddAssistantMessage(response);
    Console.WriteLine("");
}
```

尝试运行代码，和自己本地 Phi-3 模型聊聊天

以上为使用 `feiyun0112.SemanticKernel.Connectors.OnnxRuntimeGenAI.DirectML` 提供的连接，接下来尝试自己来实现与 SemanticKernel 的对接代码

在 SemanticKernel 里面定义了 IChatCompletionService 接口，以上代码的 `GetStreamingChatMessageContentsAsync` 方法功能核心就是调用 IChatCompletionService 接口提供的 GetStreamingChatMessageContentsAsync 方法

熟悉依赖注入的伙伴也许一下就看出来了，只需要注入 IChatCompletionService 接口的实现即可。在注入之前，还需要咱自己定义一个继承 IChatCompletionService 的类型，才能创建此类型进行注入

如以下代码，定义继承 IChatCompletionService 的 Phi3ChatCompletionService 类型

```csharp
class Phi3ChatCompletionService : IChatCompletionService
{
    ...
}
```

接着实现接口要求的方法，本文这里只用到 GetStreamingChatMessageContentsAsync 方法，于是就先只实现此方法

根据[上一篇](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-DirectML-%E6%8E%A7%E5%88%B6%E5%8F%B0%E8%BF%90%E8%A1%8C-Phi-3-%E6%A8%A1%E5%9E%8B.html )博客可以了解到 Phi-3 的初始化方法，先放在 Phi3ChatCompletionService 的构造函数进行初始化，代码如下

```csharp
class Phi3ChatCompletionService : IChatCompletionService
{
    public Phi3ChatCompletionService(string modelPath)
    {
        var model = new Model(modelPath);
        var tokenizer = new Tokenizer(model);

        Model = model;
        Tokenizer = tokenizer;
    }

    public IReadOnlyDictionary<string, object?> Attributes { get; set; } = new Dictionary<string, object?>();
    public Model Model { get; }
    public Tokenizer Tokenizer { get; }

    ... // 忽略其他代码
}
```

定义 GetStreamingChatMessageContentsAsync 方法代码如下

```csharp
class Phi3ChatCompletionService : IChatCompletionService
{
    ... // 忽略其他代码

    public async IAsyncEnumerable<StreamingChatMessageContent> GetStreamingChatMessageContentsAsync(ChatHistory chatHistory,
        PromptExecutionSettings? executionSettings = null, Kernel? kernel = null,
        CancellationToken cancellationToken = new CancellationToken())
    {
        ... // 忽略其他代码
    }
}
```

这里传入的是 ChatHistory 类型，咱需要进行一些提示词的转换才能让 Phi-3 更开森，转换代码如下

```csharp
        var stringBuilder = new StringBuilder();
        foreach (ChatMessageContent chatMessageContent in  chatHistory)
        {
            stringBuilder.Append($"<|{chatMessageContent.Role}|>\n{chatMessageContent.Content}");
        }
        stringBuilder.Append("<|end|>\n<|assistant|>");

        var prompt = stringBuilder.ToString();
```

完成之后，即可构建输入，以及调用 ComputeLogits 等方法，代码如下

```csharp
    public async IAsyncEnumerable<StreamingChatMessageContent> GetStreamingChatMessageContentsAsync(ChatHistory chatHistory,
        PromptExecutionSettings? executionSettings = null, Kernel? kernel = null,
        CancellationToken cancellationToken = new CancellationToken())
    {
        var stringBuilder = new StringBuilder();
        foreach (ChatMessageContent chatMessageContent in  chatHistory)
        {
            stringBuilder.Append($"<|{chatMessageContent.Role}|>\n{chatMessageContent.Content}");
        }
        stringBuilder.Append("<|end|>\n<|assistant|>");

        var prompt = stringBuilder.ToString();

        var generatorParams = new GeneratorParams(Model);

        var sequences = Tokenizer.Encode(prompt);

        generatorParams.SetSearchOption("max_length", 1024);
        generatorParams.SetInputSequences(sequences);
        generatorParams.TryGraphCaptureWithMaxBatchSize(1);

        using var tokenizerStream = Tokenizer.CreateStream();
        using var generator = new Generator(Model, generatorParams);

        while (!generator.IsDone())
        {
            var result = await Task.Run(() =>
            {
                generator.ComputeLogits();
                generator.GenerateNextToken();

                // 这里的 tokenSequences 就是在输入的 sequences 后面添加 Token 内容

                // 取最后一个进行解码为文本
                var lastToken = generator.GetSequence(0)[^1];
                var decodeText = tokenizerStream.Decode(lastToken);

                // 有些时候这个 decodeText 是一个空文本，有些时候是一个单词
                // 空文本的可能原因是需要多个 token 才能组成一个单词
                // 在 tokenizerStream 底层已经处理了这样的情况，会在需要多个 Token 才能组成一个单词的情况下，自动合并，在多个 Token 中间的 Token 都返回空字符串，最后一个 Token 才返回组成的单词
                if (!string.IsNullOrEmpty(decodeText))
                {
                    return decodeText;
                }

                return null;
            });

            if (!string.IsNullOrEmpty(result))
            {
                yield return new StreamingChatMessageContent(AuthorRole.Assistant, result);
            }
        }
    }
```

如此即可完成对接的核心代码实现，接下来只需要将 Phi3ChatCompletionService 注入即可，代码如下

```csharp
var modelPath = @"C:\lindexi\Phi3\directml-int4-awq-block-128";

// create kernel
var builder = Kernel.CreateBuilder();
builder.Services.AddSingleton<IChatCompletionService>(new Phi3ChatCompletionService(modelPath));
```

这就是完全自己实现将本地 Phi-3 模型与 SemanticKernel 进行对接的方法了，尝试运行一下项目，或者使用以下方法拉取我的代码更改掉模型文件夹，试试运行效果

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/39a65e0e6703241bdab0a836e84532bddd4385c7/SemanticKernelSamples/BemjawhufawJairkihawyawkerene) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/39a65e0e6703241bdab0a836e84532bddd4385c7/SemanticKernelSamples/BemjawhufawJairkihawyawkerene) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 39a65e0e6703241bdab0a836e84532bddd4385c7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 39a65e0e6703241bdab0a836e84532bddd4385c7
```

获取代码之后，进入 SemanticKernelSamples/BemjawhufawJairkihawyawkerene 文件夹，即可获取到源代码

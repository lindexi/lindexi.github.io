本文将和大家介绍 LLM 的魔法，通过自然语言编程的方式开发 SemanticFunction 函数

<!--more-->


<!-- CreateTime:2023/9/4 8:45:17 -->
<!-- 标题： SemanticFunction 自然语言函数 -->
<!-- 发布 -->
<!-- 博客 -->

大家都知道，编程里面的函数可以是一个完成某个功能的逻辑片段，绝大部分的函数都需要使用人类不友好的编程语言进行开发。以往只有熟练掌控和计算机沟通的编程语言的程序员才能开发出一个个功能函数，而现在有了 LLM 的魔法帮助下，人们也可以用自然语言编写出完成某项功能逻辑函数，更有趣的是使用自然语言编程写出的函数在特定领域下可以实现出更加智能容错更高的函数。如 [宵伯特](https://www.cnblogs.com/xbotter) 所说：LLM的强大之处在于可以架起自然语言与机器语言之间的桥梁。通过合适的提示词，我们可以让LLM把自然语言中的关健信息提取出来，哪怕是文本背后的一些隐含信息也可以进行处理

接下来我将和大家介绍在配合 AzureAI 的 GPT 大语言模型下的 SemanticFunction 自然语言函数的开发和对接方法

本文属于 SemanticKernel 入门系列博客，更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 或 [博客园的合集](https://www.cnblogs.com/lindexi/collections/6439)

本文开始之前，需要大家有一个 AzureAI 或 OpenAI 的账号，包括 Endpoint 和 ApiKey 等对接必备信息。否则你将不能跟随本文的演示代码进入 LLM 的世界

申请 AzureAI 地址：[https://aka.ms/oai/access](https://aka.ms/oai/access)

如等不急申请且咱认识的情况下，可以向我要一个号给你测试


## 配置

本文将演示在 .NET 9 里面使用 `1.15.1` 版本的 Microsoft.SemanticKernel 框架上的应用

在开始之前的配置需要先准备好 AzureAI 的密钥和准备好部署的模型。本文这里我已经部署完成了 GPT-4o 模型了，连接的配置代码如下

```csharp
using Microsoft.SemanticKernel;

// 这里的演示代码需要用到 AzureAI 的支持，需要提前申请好，申请地址：https://aka.ms/oai/access

var endpoint = "https://lindexi.openai.azure.com/"; // 请换成你的地址
var apiKey = args[0]; // 请换成你的密钥

var kernel = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion("GPT4o", endpoint, apiKey)
    // 当然，这里也可以支持 OpenAI 的服务。或者是其他第三方的服务
    //.WithOpenAIChatCompletionService()
    .Build();
```

使用 SemanticKernel 的强大之处在于能够支持许多其他 AI 服务，如 OpenAI 的服务或其他第三方的服务，甚至本地小语言模型。更多关于 SemanticKernel 与本地小语言模型对接，请参阅 [dotnet 将本地的 Phi-3 模型与 SemanticKernel 进行对接](https://blog.lindexi.com/post/dotnet-%E5%B0%86%E6%9C%AC%E5%9C%B0%E7%9A%84-Phi-3-%E6%A8%A1%E5%9E%8B%E4%B8%8E-SemanticKernel-%E8%BF%9B%E8%A1%8C%E5%AF%B9%E6%8E%A5.html )
<!-- [dotnet 将本地的 Phi-3 模型与 SemanticKernel 进行对接 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18257888 ) -->

关于 AzureAI 的模型部署，可以在网上搜到非常多大佬的博客，我这里就不进行展开。相信大家看 Azure 界面也就知道如何配置了

## 定义自然语言函数

接下来的编程可就是用自然语言开始编写了。本文的例子是做一个帮忙找借口的函数，这个函数的作用就是你输入做了什么事情，然后 GPT 帮你找一个夸张的借口。这个例子是原本 SemanticKernel 官方的例子，我只是将其修改为中文

先使用自然语言定义出函数，这里采用 `$input` 作为变量的占位符号，这是 [上一篇博客 自定义变量和技能](https://blog.lindexi.com/post/dotnet-SemanticKernel-%E5%85%A5%E9%97%A8-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%8F%98%E9%87%8F%E5%92%8C%E6%8A%80%E8%83%BD.html ) 里所提到的方法。换成使用其他的变量占位符也是可以的

```csharp
const string FunctionDefinition = @"
为给定的事件想出一个创造性的理由或借口。
要有创意，要有趣。让你的想象力尽情驰骋。

事情：我要迟到了。
借口：我被长颈鹿帮绑架了。

事情：我有一年没去健身房了
借口：我一直忙着训练我的宠物龙。

事情： { {$input} }
借口：";
```

以上的函数就是先给 GPT 两个例子，然后第三个就是让 GPT 进行自由发挥

以上的代码里面，我将连续两个花括号 `{` 中间使用空格分开，这只是为了让我的博客转换器开森而已，实际使用还请使用下文贴出的代码仓库里面的代码为准

完成了自然语言函数编写之后，接下来就可以和 SemanticKernel 框架进行对接了

更多关于 Prompt 提示词相关，请参阅 [简介 - Learning Prompt](https://learningprompt.wiki/zh-Hans/docs/chatGPT/tutorial-basics/brief-introduction )

## 对接框架

对接框架最简单的方法就是通过 CreateFunctionFromPrompt 方法创建出 KernelFunction 函数，如以下代码

```csharp
KernelFunction kernelFunction = kernel.CreateFunctionFromPrompt(FunctionDefinition);
```

这里的代码与预览版 `0.20.230821.4-preview` 版本的 SemanticKernel 的差异比较大，还请大家在更新版本时，将此进行替换

完成函数的制作之后，接下来咱来看看怎么调用吧

## 调用函数

调用的时候，既可以使用 Kernel 发起，也可以使用 KernelFunction 发起。在发起之前需要先准备好参数，如以下代码，我将创建一个参数

```csharp
var kernelArguments = new KernelArguments();
kernelArguments.Add("input", "我错过了篮球赛");
```

这里的参数的作用就是将以上的提示词的占位符替换为实际业务的输入，大家可以自行发挥输入一些更加有趣的事情，看看 GPT 是如何编借口

完成参数之后，即可使用 InvokeAsync 方法进行调用，代码如下

```csharp
var result = await kernelFunction.InvokeAsync(kernel, kernelArguments);
```

这里返回的 result 是一个 FunctionResult 类型，可以直接使用 ToString 方法获取其结果字符串，如下面代码进行输出

```csharp
Console.WriteLine(result);
```

我跑了一次，输出了以下代码

```csharp
我被邀请参加了一个秘密的超级英雄训练营，我必须去拯救世界！
```

看起来 GPT 的这个函数算是定义成功了，但就是回答有些奇怪，还需要继续炼丹，修改自然语言

也许后面会有许多面向 LLM 的开发者，这些开发者不再编写传统的编程语言的代码，而是编写自然语言，让 LLM 实现魔法的功能

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/027b53ffdf952546342ba0c6c474b3e06d736b65/SemanticKernelSamples/Example05_InlineFunctionDefinition) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/027b53ffdf952546342ba0c6c474b3e06d736b65/SemanticKernelSamples/Example05_InlineFunctionDefinition) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 027b53ffdf952546342ba0c6c474b3e06d736b65
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 027b53ffdf952546342ba0c6c474b3e06d736b65
```

获取代码之后，进入 SemanticKernelSamples/Example05_InlineFunctionDefinition 文件夹，即可获取到源代码

---

以下为预览版 `0.20.230821.4-preview` 版本的 SemanticKernel 的用法

## 配置

在 KernelBuilder 里面可以通过 WithAzureChatCompletionService 或 WithAzureTextCompletionService 方法进行配置和 AzureAI 的对接。我这里申请的是 GPT 3.5 16K 的，只支持 ChatCompletion 方式，因此就选 WithAzureChatCompletionService 进行配置

需要传入的是部署的模型和 Endpoint 和 ApiKey 这两个必要的信息，如以下代码进行配置

```csharp
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;

var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddConsole();
    builder.SetMinimumLevel(LogLevel.Debug);
});
var logger = loggerFactory.CreateLogger("SemanticKernel");

// 这里的演示代码需要用到 AzureAI 的支持，需要提前申请好，申请地址：https://aka.ms/oai/access

var endpoint = "https://lindexi.openai.azure.com/"; // 请换成你的地址
var apiKey = args[0]; // 请换成你的密钥

IKernel kernel = new KernelBuilder()
    .WithLogger(logger)
    .WithAzureChatCompletionService("GPT35", endpoint, apiKey)
    // 当然，这里也可以支持 OpenAI 的服务。或者是其他第三方的服务
    //.WithOpenAIChatCompletionService()
    .Build();
```

以上代码还在配置的时候注入控制台日志，方便出错时了解错误原因

关于 AzureAI 的模型部署，可以在网上搜到非常多大佬的博客，我这里就不进行展开。相信大家看 Azure 界面也就知道如何配置了

如果拿到的是 OpenAI 的服务，则可以选用 WithOpenAIChatCompletionService 或 WithOpenAITextCompletionService 进行配置

## 定义自然语言函数

接下来的编程可就是用自然语言开始编写了。本文的例子是做一个帮忙找借口的函数，这个函数的作用就是你输入做了什么事情，然后 GPT 帮你找一个夸张的借口。这个例子是原本 SemanticKernel 官方的例子，我只是将其修改为中文

先使用自然语言定义出函数，这里采用 `$input` 作为变量的占位符号，这是 [上一篇博客 自定义变量和技能](https://blog.lindexi.com/post/dotnet-SemanticKernel-%E5%85%A5%E9%97%A8-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%8F%98%E9%87%8F%E5%92%8C%E6%8A%80%E8%83%BD.html ) 里所提到的方法

```csharp
const string FunctionDefinition = @"
为给定的事件想出一个创造性的理由或借口。
要有创意，要有趣。让你的想象力尽情驰骋。

事情：我要迟到了。
借口：我被长颈鹿帮绑架了。

事情：我有一年没去健身房了
借口：我一直忙着训练我的宠物龙。

事情： { {$input} }
借口：";
```

以上的函数就是先给 GPT 两个例子，然后第三个就是让 GPT 进行自由发挥

完成了自然语言函数编写之后，接下来就可以和 SemanticKernel 框架进行对接了

更多关于 Prompt 提示词相关，请参阅 [简介 - Learning Prompt](https://learningprompt.wiki/zh-Hans/docs/chatGPT/tutorial-basics/brief-introduction )

## 对接框架

对接框架最简单的方法就是通过 CreateSemanticFunction 方法创建出 SemanticFunction 函数，如以下代码

```csharp
ISKFunction excuseFunction = kernel.CreateSemanticFunction(FunctionDefinition, maxTokens: 200,
    // 温度高一些，这样 GPT 才会乱说
    temperature: 1);
```

这时候为了让 GPT 能够回答更加有趣，这里提升了温度

完成函数的制作之后，接下来咱来看看怎么调用吧

## 调用函数

由于 CreateSemanticFunction 方法返回的是一个 ISKFunction 接口，也就是和前面博客提到的技能的函数在使用上没有多少的不同，从这里也可以看出 SemanticKernel 故意模糊了传统的编程函数和自然语言函数，从而让大家在调用的时候不需要关注某个函数是如何创建的

```csharp
var result = await excuseFunction.InvokeAsync("我错过了篮球赛");
Console.WriteLine(result);
```

我跑了一次，输出了以下代码

```csharp
我被邀请参加了一个秘密的超级英雄训练营，我必须去拯救世界！
```

看起来 GPT 的这个函数算是定义成功了，但就是回答有些奇怪，还需要继续炼丹，修改自然语言

也许后面会有许多面向 LLM 的开发者，这些开发者不再编写传统的编程语言的代码，而是编写自然语言，让 LLM 实现魔法的功能

## 代码

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/27492e28eee195378642ec34e3cc8d9d9e1417a9/SemanticKernelSamples/Example05_InlineFunctionDefinition) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/27492e28eee195378642ec34e3cc8d9d9e1417a9/SemanticKernelSamples/Example05_InlineFunctionDefinition) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 27492e28eee195378642ec34e3cc8d9d9e1417a9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 27492e28eee195378642ec34e3cc8d9d9e1417a9
```

获取代码之后，进入 SemanticKernelSamples\Example05_InlineFunctionDefinition 文件夹

## 参考文档

[Semantic Kernel 入门系列：LLM的魔法 - 宵伯特 - 博客园](https://www.cnblogs.com/xbotter/p/semantic_kernel_introduction_llm_magic.html)

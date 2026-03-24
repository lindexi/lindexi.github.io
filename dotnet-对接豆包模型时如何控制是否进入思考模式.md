
# dotnet 对接豆包模型时如何控制是否进入思考模式

本文记录使用 OpenAI 的 API 和豆包进行对接的时候，如何控制豆包是否进入思考模式。本文内容可直接与 Microsoft Agent Framework 对接

<!--more-->


<!-- CreateTime:2026/03/24 08:49:42 -->

<!-- 发布 -->
<!-- 博客 -->

按照豆包官方文档可以知道，通过 `thinking` 的类型可以控制豆包是否进入思考模式，官方文档请看： <https://www.volcengine.com/docs/82379/1449737>

可选参数如下：

- enabled：强制开启，强制开启深度思考能力。
- disabled：强制关闭深度思考能力。
- auto：模型自行判断是否进行深度思考

按照 GitHub 上的 <https://github.com/openai/openai-dotnet/issues/132> 可了解到对应的 OpenAI 的调用方法是通过 ChatCompletionOptions 的 Patch 属性设置，代码如下

```csharp
var chatCompletionOptions = new ChatCompletionOptions();

#pragma warning disable SCME0001

// https://www.volcengine.com/docs/82379/1449737
// 提供 thinking 字段控制是否关闭深度思考能力，实现“复杂任务深度推理，简单任务高效响应”的精细控制，获得成本、效率收益
// 取值说明：
// enabled：强制开启，强制开启深度思考能力。
// disabled：强制关闭深度思考能力。
// auto：模型自行判断是否进行深度思考。
chatCompletionOptions.Patch.Set("$.thinking"u8, BinaryData.FromString("""{ "type": "disabled" }"""));
```

这里需要明确使用 `#pragma warning disable SCME0001` 开启实验性功能。这是因为 Patch 属性现在官方还没考虑好，还不确定是否如此开放，因此被标记了实验性功能

在调用 CompleteChatStreamingAsync 方法的时候，将以上的 ChatCompletionOptions 传递进去就可以了，代码如下

```csharp
await foreach (var streamingChatCompletionUpdate in chatClient.CompleteChatStreamingAsync([new UserChatMessage("你好")], chatCompletionOptions))
{
}
```

以上方法可以配合在 Microsoft Agent Framework 的 `Microsoft.Agents.AI.OpenAI` 库使用，对应调用豆包的 `https://ark.cn-beijing.volces.com/api/v3` 地址即可

全部演示代码如下

```csharp
using OpenAI;
using OpenAI.Chat;

using System;
using System.ClientModel;
using System.ClientModel.Primitives;
using System.Text;
using System.Text.Json;

var keyFile = @"C:\lindexi\Work\Doubao.txt";
var key = File.ReadAllText(keyFile); // 请换成你自己的 key 哦

var openAiClient = new OpenAIClient(new ApiKeyCredential(key), new OpenAIClientOptions()
{
    Endpoint = new Uri("https://ark.cn-beijing.volces.com/api/v3"),

});

var chatClient = openAiClient.GetChatClient("ep-20260306101224-c8mtg"); // 请换成你自己的模型
var chatCompletionOptions = new ChatCompletionOptions();
#pragma warning disable SCME0001

// https://www.volcengine.com/docs/82379/1449737
// 提供 thinking 字段控制是否关闭深度思考能力，实现“复杂任务深度推理，简单任务高效响应”的精细控制，获得成本、效率收益
// 取值说明：
// enabled：强制开启，强制开启深度思考能力。
// disabled：强制关闭深度思考能力。
// auto：模型自行判断是否进行深度思考。
chatCompletionOptions.Patch.Set("$.thinking"u8, BinaryData.FromString("""{ "type": "disabled" }"""));

#pragma warning restore SCME0001

bool isThinking = true;
bool isAnyOutput = false;
bool isAnyThinking = false;
bool isFirstTextOutput = true;

await foreach (var streamingChatCompletionUpdate in chatClient.CompleteChatStreamingAsync([new UserChatMessage("你好")], chatCompletionOptions))
{
    var chatMessageContent = streamingChatCompletionUpdate.ContentUpdate;

#pragma warning disable SCME0001
    ref JsonPatch patch = ref streamingChatCompletionUpdate.Patch;
#pragma warning restore SCME0001

    if (patch.TryGetJson("$.choices[0].delta"u8, out var data))
    {
        var jsonElement = JsonElement.Parse(data.Span);
        if (jsonElement.TryGetProperty("reasoning_content", out var reasoningContent))
        {
            if (!isAnyOutput && isThinking)
            {
                Console.WriteLine("思考：");
            }

            isAnyThinking = true;
            Console.Write(reasoningContent.ToString());
        }
    }

    foreach (var chatMessageContentPart in chatMessageContent)
    {
        if (!string.IsNullOrEmpty(chatMessageContentPart.Text))
        {
            isThinking = false;

            if (isAnyThinking && isFirstTextOutput)
            {
                // 有思考，且当前是首次内容输出，输出分隔符
                Console.WriteLine();
                Console.WriteLine("---------");
            }

            isFirstTextOutput = false;
        }

        Console.Write(chatMessageContentPart.Text);
    }

    isAnyOutput = true;
}

Console.WriteLine();

Console.Read();
```

请自行尝试切换 `thinking` 的模式，运行测试控制台输出内容

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7cf5cc06e00b03ec7d695911ae6bfdc1ed9701d8/SemanticKernelSamples/JunayfallruHaldearwaije) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7cf5cc06e00b03ec7d695911ae6bfdc1ed9701d8/SemanticKernelSamples/JunayfallruHaldearwaije) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7cf5cc06e00b03ec7d695911ae6bfdc1ed9701d8
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7cf5cc06e00b03ec7d695911ae6bfdc1ed9701d8
```

获取代码之后，进入 SemanticKernelSamples/JunayfallruHaldearwaije 文件夹，即可获取到源代码

相关博客：

- [C# Microsoft Agent Framework 与 豆包 对接](https://blog.lindexi.com/post/C-Microsoft-Agent-Framework-%E4%B8%8E-%E8%B1%86%E5%8C%85-%E5%AF%B9%E6%8E%A5.html )
<!-- [C# Microsoft Agent Framework 与 豆包 对接 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19494917 ) -->

- [Microsoft Agent Framework 取出 DeepSeek 思考内容](https://blog.lindexi.com/post/Microsoft-Agent-Framework-%E5%8F%96%E5%87%BA-DeepSeek-%E6%80%9D%E8%80%83%E5%86%85%E5%AE%B9.html )
<!-- [Microsoft Agent Framework 取出 DeepSeek 思考内容 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19635618 ) -->




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
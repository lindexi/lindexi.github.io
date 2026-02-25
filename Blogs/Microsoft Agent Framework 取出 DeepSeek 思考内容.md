---
title: Microsoft Agent Framework 取出 DeepSeek 思考内容
description: 默认情况下的 Microsoft Agent Framework 是没有将 DeepSeek 思考内容提供出来的。官方给出的原因是当前 OpenAI 提供的 API 还没有将 reasoning_content 放开。好在 Microsoft Agent Framework 也没有堵住获取思考内容的获取方法，咱依然可以从返回结果获取思考内容
tags: 
category: 
---

<!-- CreateTime:2026/02/25 07:21:57 -->

<!-- 发布 -->
<!-- 博客 -->

本文提供的方法适用于 DeepSeek 和豆包等模型

前置博客：

- [Microsoft Agent Framework 与 DeepSeek 对接](https://blog.lindexi.com/post/Microsoft-Agent-Framework-%E4%B8%8E-DeepSeek-%E5%AF%B9%E6%8E%A5.html )
<!-- [Microsoft Agent Framework 与 DeepSeek 对接 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19413475 ) -->

- [C# Microsoft Agent Framework 与 豆包 对接](https://blog.lindexi.com/post/C-Microsoft-Agent-Framework-%E4%B8%8E-%E8%B1%86%E5%8C%85-%E5%AF%B9%E6%8E%A5.html )
<!-- [C# Microsoft Agent Framework 与 豆包 对接 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19494917 ) -->

核心原理是从 AgentResponseUpdate 里面的 RawRepresentation 获取 reasoning_content 字段

核心代码如下

```csharp
AIAgent agent = ...;
IEnumerable<ChatMessage> messages = ...;
AgentSession? session = ...;
AgentRunOptions? options = ...;

        await foreach (AgentResponseUpdate agentRunResponseUpdate in agent.RunStreamingAsync(messages, session, options, cancellationToken))
        {
            var contentIsEmpty = string.IsNullOrEmpty(agentRunResponseUpdate.Text);

            if (contentIsEmpty && agentRunResponseUpdate.RawRepresentation is Microsoft.Extensions.AI.ChatResponseUpdate streamingChatCompletionUpdate)
            {
                if (streamingChatCompletionUpdate.RawRepresentation is StreamingChatCompletionUpdate chatCompletionUpdate)
                {
#pragma warning disable SCME0001 // Patch 属性是实验性内容
                    ref JsonPatch patch = ref chatCompletionUpdate.Patch;
                    if (patch.TryGetJson("$.choices[0].delta"u8, out var data))
                    {
                        var jsonElement = JsonElement.Parse(data.Span);
                        if (jsonElement.TryGetProperty("reasoning_content", out var reasoningContent))
                        {
                            // 拿到的 reasoningContent 就是思考内容
                        }
                    }

#pragma warning restore SCME0001
                }
            }
```

我将这段代码封装为扩展方法，方便上层业务使用，代码如下

```csharp
using System.ClientModel.Primitives;
using System.Diagnostics;
using System.Runtime.CompilerServices;
using System.Text.Json;
using OpenAI.Chat;
using ChatMessage = Microsoft.Extensions.AI.ChatMessage;

namespace Microsoft.Agents.AI.Reasoning;

public static class ReasoningAIAgentExtension
{
    public static IAsyncEnumerable<ReasoningAgentRunResponseUpdate> RunReasoningStreamingAsync(this AIAgent agent, ChatMessage message,
        AgentSession? session = null,
        AgentRunOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        return RunReasoningStreamingAsync(agent, [message], session, options, cancellationToken);
    }

    public static async IAsyncEnumerable<ReasoningAgentRunResponseUpdate> RunReasoningStreamingAsync(this AIAgent agent, IEnumerable<ChatMessage> messages,
        AgentSession? session = null,
        AgentRunOptions? options = null,
        [EnumeratorCancellation]
        CancellationToken cancellationToken = default)
    {
        bool? isThinking = null;
        bool isFirstOutputContent = true;

        await foreach (AgentResponseUpdate agentRunResponseUpdate in agent.RunStreamingAsync(messages, session, options, cancellationToken))
        {
            var contentIsEmpty = string.IsNullOrEmpty(agentRunResponseUpdate.Text);

            if (contentIsEmpty && agentRunResponseUpdate.RawRepresentation is Microsoft.Extensions.AI.ChatResponseUpdate streamingChatCompletionUpdate)
            {
                if (streamingChatCompletionUpdate.RawRepresentation is StreamingChatCompletionUpdate chatCompletionUpdate)
                {
#pragma warning disable SCME0001 // Patch 属性是实验性内容
                    ref JsonPatch patch = ref chatCompletionUpdate.Patch;
                    if (patch.TryGetJson("$.choices[0].delta"u8, out var data))
                    {
                        var jsonElement = JsonElement.Parse(data.Span);
                        if (jsonElement.TryGetProperty("reasoning_content", out var reasoningContent))
                        {
                            // 拿到的 reasoningContent 就是思考内容
                        }
                    }

#pragma warning restore SCME0001
                }
            }

            if (!contentIsEmpty)
            {
                var responseUpdate = new ReasoningAgentRunResponseUpdate(agentRunResponseUpdate);

                if (isFirstOutputContent)
                {
                    responseUpdate.IsFirstOutputContent = true;
                }

                if (isThinking is true && isFirstOutputContent)
                {
                    responseUpdate.IsThinkingEnd = true;
                }

                isFirstOutputContent = false;
                isThinking = false;

                yield return responseUpdate;
            }
        }
    }
}
```

用到的辅助类 ReasoningAgentRunResponseUpdate 代码如下

```csharp
namespace Microsoft.Agents.AI.Reasoning;

public class ReasoningAgentRunResponseUpdate : AgentResponseUpdate
{
    public ReasoningAgentRunResponseUpdate(AgentResponseUpdate origin) : base(origin.Role, origin.Contents)
    {
        Origin = origin;
        AdditionalProperties = origin.AdditionalProperties;
        AuthorName = origin.AuthorName;
        CreatedAt = origin.CreatedAt;
        MessageId = origin.MessageId;
        RawRepresentation = origin.RawRepresentation;
        ResponseId = origin.ResponseId;
        ContinuationToken = origin.ContinuationToken;
        AgentId = origin.AgentId;
    }

    public AgentResponseUpdate Origin { get; }

    public string? Reasoning { get; set; }

    /// <summary>
    /// 是否首次输出内容，前面输出的都是内容
    /// </summary>
    /// 仅内容输出，无思考的首次内容输出：
    /// - IsFirstOutputContent = true
    /// - IsFirstThinking = false
    /// - IsThinkingEnd = false
    /// 有思考，完成思考后的首次内容输出：
    /// - IsFirstOutputContent = true
    /// - IsFirstThinking = false
    /// - IsThinkingEnd = true
    public bool IsFirstOutputContent { get; set; }

    /// <summary>
    /// 思考的首次输出
    /// </summary>
    public bool IsFirstThinking { get; set; }

    /// <summary>
    /// 是否思考结束
    /// </summary>
    public bool IsThinkingEnd { get; set; }
}
```

业务层使用示例：

```csharp
ChatClientAgent aiAgent = ...;

ChatMessage message = new ChatMessage(ChatRole.User, "请讲一个笑话");

await foreach (var agentRunResponseUpdate in aiAgent.RunReasoningStreamingAsync(message))
{
    if (agentRunResponseUpdate.IsFirstThinking)
    {
        Console.WriteLine("思考：");
    }

    if (agentRunResponseUpdate.Reasoning is not null)
    {
        Console.Write(agentRunResponseUpdate.Reasoning);
    }

    if (agentRunResponseUpdate.IsThinkingEnd)
    {
        Console.WriteLine();
        Console.WriteLine("--------");
    }

    var text = agentRunResponseUpdate.Text;
    if (!string.IsNullOrEmpty(text))
    {
        Console.Write(text);
    }
}
```

输出效果如下：

```
思考：
用户让我讲一个笑话，那得选个轻松有趣、贴近生活的，别太复杂。首先，得考虑不同年龄段都能听懂，最好有点小反转，这样笑点更足。

比如可以从日常场景入手，像去超市、和朋友互动，或者宠物相关的？宠物的笑话通常很受欢迎，因为可爱又有反差感。

哦，想到一个关于猫和主人的：

主人发现猫最近总偷喝鱼缸里的水，于是教训它：“你再偷喝，我就把你和鱼一起炖了！”
第二天，主人发现鱼缸里的鱼全没了，猫蹲在旁边舔爪子，抬头一脸无辜：“主人，我帮你提前处理了食材～”

这个有小反转，猫的逻辑很可爱，应该挺逗的。或者换个更接地气的，比如关于吃饭的：

昨天去吃火锅，服务员问我要辣的还是不辣的，我说：“微辣就行，别太辣。”
服务员：“好的，我们这边微辣是‘外地人眼中的特辣’，特辣是‘本地人眼中的微辣’，您要哪个？”
我：“……给我来个‘外地人眼中的微辣’的微辣吧。”
服务员愣了一下：“好的，清水锅加一片辣椒！”

这个也很真实，很多地方的微辣都很辣，有共鸣感。不过还是宠物的那个更可爱，可能用户会更喜欢。再调整一下，让语言更自然：

有只猫特别爱偷喝鱼缸里的水，主人每次看见都批评它：“再偷喝，我就把你和鱼一起炖了！”
结果第二天起床，主人发现鱼缸里的鱼全不见了，猫正蹲在旁边舔嘴巴，看见主人过来，还一脸邀功地说：“主人你看！我提前把配菜都处理好了??”

对，这样更生动，有表情符号也更活泼。或者再短一点的？比如冷笑话？不过可能用户想要正常的笑话，不是冷的。那就选这个猫的吧，应该不错。
--------
有只猫特别爱偷喝鱼缸里的水，主人每次看见都批评它：“再偷喝，我就把你和鱼一起炖了！”

结果第二天起床，主人发现鱼缸里的鱼全不见了，猫正蹲在旁边舔嘴巴，看见主人过来，还一脸邀功地说：“主人你看！我提前把配菜都处理好了”
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/fc4cf4f485ea3e2268a67c0c6900827d9803d9b3/SemanticKernelSamples/LadelallkeacheWhikurwearqobakaju) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/fc4cf4f485ea3e2268a67c0c6900827d9803d9b3/SemanticKernelSamples/LadelallkeacheWhikurwearqobakaju) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin fc4cf4f485ea3e2268a67c0c6900827d9803d9b3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin fc4cf4f485ea3e2268a67c0c6900827d9803d9b3
```

获取代码之后，进入 SemanticKernelSamples/LadelallkeacheWhikurwearqobakaju 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

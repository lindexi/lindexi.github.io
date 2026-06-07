# Microsoft Agent Framework 中 RequirePerServiceCallChatHistoryPersistence 对 ReduceAsync 调用时机的影响

本文将演示在 Microsoft Agent Framework 中，RequirePerServiceCallChatHistoryPersistence 选项对 ChatReducer 的 ReduceAsync 方法调用时机的影响，并说明在不同场景下应该如何选择开启或关闭该选项。

<!--more-->
<!-- CreateTime:2026/06/03 07:04:01 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 主导编写，人类只做简单审查

本文写于 2026年6月2日，现在的 RequirePerServiceCallChatHistoryPersistence 还是实验性功能，未来行为可能变更

## 背景

在 Microsoft Agent Framework 中，ChatHistoryProvider 负责存储和管理对话历史。随着对话轮次的增加，聊天上下文会越来越长，最终可能超出 LLM 的 token 限制。为了解决这个问题，框架提供了 IChatReducer 接口，允许开发者在上下文过长时对消息列表进行裁剪，只保留最关键的部分。

ChatReducer 的核心方法是 ReduceAsync，它接收当前完整的聊天消息列表，返回裁剪后的消息列表。那么问题来了：ReduceAsync 在什么时机会被调用？是一次对话只调用一次，还是在每次 LLM 服务调用（工具调用）后都会触发？如果想要在 Microsoft Agent Framework 里面在每次工具调用之后，都触发压缩上下文动作，可以怎么办？

答案取决于 RequirePerServiceCallChatHistoryPersistence 选项的设置。

## 核心概念

RequirePerServiceCallChatHistoryPersistence 是 ChatClientAgentOptions 中的一个布尔选项，默认值为 false。它控制的是：在一次完整的 Agent 运行过程中，是否在每一次 LLM 服务调用完成后都立即持久化聊天历史。

持久化聊天历史的流程中包含了调用 ReduceAsync。因此，这个选项实际上间接控制了 ReduceAsync 的调用频率：

- 设置为 true：每次 LLM 服务调用完成后，都会触发一次 ReduceAsync。这意味着在多轮工具调用场景中，每完成一次工具调用往返，上下文就会被裁剪一次，有效防止中间消息的堆积。
- 设置为 false（默认值）：ReduceAsync 只在对话流程的初始阶段被调用，不会在每次工具调用后触发。这意味着在多轮工具调用的过程中，中间消息会持续累积。

为了更好地演示这一行为差异，下面我们搭建一套可控的演示环境。

## 搭建演示环境

真实 LLM 的工具调用行为存在不确定性，不便于精确对比。因此，本演示使用一个自定义的 FakeChatClient 来模拟 LLM，精确控制返回的工具调用序列。

### FakeChatClient 的设计

FakeChatClient 实现了 IChatClient 接口，其核心思路是通过委托注入响应流，使得测试代码可以完全控制 LLM 返回的内容：

```csharp
public sealed class FakeChatClient : IChatClient
{
    public Func<IEnumerable<ChatMessage>, ChatOptions?, CancellationToken, IAsyncEnumerable<ChatResponseUpdate>>?
        OnGetStreamingResponseAsync
    { get; set; }

    public IAsyncEnumerable<ChatResponseUpdate> GetStreamingResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        if (OnGetStreamingResponseAsync is null)
        {
            throw new InvalidOperationException(
                $"{nameof(FakeChatClient)}.{nameof(OnGetStreamingResponseAsync)} has not been configured.");
        }

        return OnGetStreamingResponseAsync(messages, options, cancellationToken);
    }
    // ... 其他接口成员的委托注入实现
}
```

FakeChatClient 为 IChatClient 的每个核心方法都暴露了一个可设置的委托属性。当外部调用这些方法时，实际上执行的是委托。这种设计在单元测试和演示场景中非常有用——你可以完全控制 ChatClient 的行为，而不需要 Mock 框架。

对于非流式调用、GetService 和 Dispose 等方法，FakeChatClient 也提供了对应的委托注入点，确保整个类在作为 IChatClient 使用时行为完整。

### 模拟工具调用序列

为了让演示更贴近真实场景，我们设计了一个会触发两次工具调用的 FakeChatClient。核心逻辑在 CreateToolCallingFakeChatClient 方法中：

```csharp
private static FakeChatClient CreateToolCallingFakeChatClient(int maxToolCallRounds)
{
    var callCount = 0;

    var fakeChatClient = new FakeChatClient()
    {
        OnGetStreamingResponseAsync = (messages, options, cancellationToken) => CreateResponseSequenceAsync()
    };

    return fakeChatClient;

    async IAsyncEnumerable<ChatResponseUpdate> CreateResponseSequenceAsync()
    {
        var currentCall = Interlocked.Increment(ref callCount);

        if (currentCall <= maxToolCallRounds)
        {
            var toolName = currentCall == 1 ? nameof(GetWeather) : nameof(GetTime);
            var callId = $"call_{currentCall}";

            yield return new ChatResponseUpdate(ChatRole.Assistant, new List<AIContent>()
            {
                new FunctionCallContent(callId, toolName, new Dictionary<string, object?>()
                {
                    { "location", "深圳" }
                })
            })
            {
                FinishReason = ChatFinishReason.ToolCalls,
            };
        }
        else
        {
            yield return new ChatResponseUpdate(ChatRole.Assistant,
                "根据查询结果，今天深圳是晴天，气温 25°C，现在时间是 " + DateTime.Now.ToString("HH:mm:ss") + "。")
            {
                FinishReason = ChatFinishReason.Stop,
            };
        }
    }
}
```

这段代码的设计思路如下：

1. 使用闭包变量 callCount 维护调用计数，配合 Interlocked.Increment 确保线程安全。
2. 前 maxToolCallRounds 次（本演示中为 2 次）LLM 请求返回 FinishReason 为 ToolCalls 的响应，分别触发 GetWeather 和 GetTime 工具调用。
3. 第 3 次及以后的 LLM 请求返回 FinishReason 为 Stop 的最终文本回复。

这样，一次用户输入会触发如下链路：用户消息 → LLM 请求返回"调用 GetWeather"→ 框架执行 GetWeather → LLM 请求返回"调用 GetTime"→ 框架执行 GetTime → LLM 请求返回最终文本。

### 两个模拟工具方法

GetWeather 和 GetTime 是两个简单的模拟工具方法，在控制台输出调用信息后返回固定的字符串：

```csharp
private static string GetWeather()
{
    Console.WriteLine($"  [工具调用] GetWeather 被执行，返回：晴天 25°C");
    return "晴天，25°C";
}

private static string GetTime()
{
    var time = DateTime.Now.ToString("HH:mm:ss");
    Console.WriteLine($"  [工具调用] GetTime 被执行，返回：{time}");
    return time;
}
```

这两个工具方法通过 AIFunctionFactory.Create 注册到 Agent 的 ChatOptions.Tools 列表中。框架会在收到 FinishReason 为 ToolCalls 的响应后自动查找并执行对应的工具方法。

### LoggingChatReducer 的设计

为了观察 ReduceAsync 的调用时机，我们实现了一个带日志的 IChatReducer：

```csharp
sealed class LoggingChatReducer : IChatReducer
{
    private readonly string _name;

    public LoggingChatReducer(string name)
    {
        _name = name;
    }

    public int ReduceCallCount { get; private set; }

    public Task<IEnumerable<ChatMessage>> ReduceAsync(IEnumerable<ChatMessage> messages,
        CancellationToken cancellationToken)
    {
        var messageList = messages.ToList();
        ReduceCallCount++;

        Console.WriteLine($"  [{_name}] ReduceAsync 第 {ReduceCallCount} 次被调用，当前消息数：{messageList.Count}");

        if (messageList.Count > 6)
        {
            var reduced = messageList.TakeLast(4).ToList();
            Console.WriteLine($"  [{_name}]   上下文消息数从 {messageList.Count} 裁剪为 {reduced.Count}");
            return Task.FromResult<IEnumerable<ChatMessage>>(reduced);
        }

        return Task.FromResult<IEnumerable<ChatMessage>>(messageList);
    }
}
```

LoggingChatReducer 在两个维度上记录信息：

1. 调用次数：通过 ReduceCallCount 属性累计，可供最终对比。
2. 裁剪行为：当消息数超过 6 条时，只保留最近 4 条，并输出裁剪前后消息数量的变化。

裁剪策略这里采用的是简单的"保留最近 N 条"，实际项目中你可能需要更智能的策略，比如保留 system message 的同时裁剪历史对话，或者基于 token 计数而非消息条数来判断。

## 演示对比

Main 方法依次运行两个演示，通过相同的 FakeChatClient 和不同的 RequirePerServiceCallChatHistoryPersistence 设置，直观对比 ReduceAsync 的调用频率差异。

### 演示 1：启用 Persistence

在 DemonstrateWithPersistenceAsync 方法中，创建 Agent 时显式设置 RequirePerServiceCallChatHistoryPersistence 为 true：

```csharp
var agent = fakeChatClient.AsAIAgent(new ChatClientAgentOptions()
{
    ChatHistoryProvider = new InMemoryChatHistoryProvider(new InMemoryChatHistoryProviderOptions()
    {
        ChatReducer = loggingReducer
    }),
    ChatOptions = new ChatOptions()
    {
        Tools =
        [
            AIFunctionFactory.Create(GetWeather, nameof(GetWeather)),
            AIFunctionFactory.Create(GetTime, nameof(GetTime)),
        ],
    },
    RequirePerServiceCallChatHistoryPersistence = true,
});
```

InMemoryChatHistoryProvider 是框架内置的内存聊天历史提供程序，ChatReducer 属性用于指定裁剪器实例。

运行后，控制台会显示类似如下的输出：

```
  [演示1-Reducer] ReduceAsync 第 1 次被调用，当前消息数：4
  [工具调用] GetWeather 被执行，返回：晴天 25°C
  [演示1-Reducer] ReduceAsync 第 2 次被调用，当前消息数：6
  [工具调用] GetTime 被执行，返回：15:30:42
  [演示1-Reducer] ReduceAsync 第 3 次被调用，当前消息数：8
  [演示1-Reducer]   上下文消息数从 8 裁剪为 4
```

可以看到，ReduceAsync 共计被调用了 3 次：初始阶段 1 次，每次工具调用完成后各 1 次。在第三次调用时，消息数量已经超过了 6 条的阈值，触发了裁剪。

这意味着启用 Persistence 后，每次工具调用返回的结果和下一轮 LLM 响应都会被及时"收拢"，防止上下文在多次工具调用中无节制地增长。

### 演示 2：不启用 Persistence（默认）

在 DemonstrateWithoutPersistenceAsync 方法中，不设置 RequirePerServiceCallChatHistoryPersistence（保持默认 false）：

```csharp
var agent = fakeChatClient.AsAIAgent(new ChatClientAgentOptions()
{
    ChatHistoryProvider = new InMemoryChatHistoryProvider(new InMemoryChatHistoryProviderOptions()
    {
        ChatReducer = loggingReducer
    }),
    ChatOptions = new ChatOptions()
    {
        Tools =
        [
            AIFunctionFactory.Create(GetWeather, nameof(GetWeather)),
            AIFunctionFactory.Create(GetTime, nameof(GetTime)),
        ],
    },
    // RequirePerServiceCallChatHistoryPersistence 默认为 false，不设置
});
```

运行后，控制台输出如下：

```
  [演示2-Reducer] ReduceAsync 第 1 次被调用，当前消息数：3
  [工具调用] GetWeather 被执行，返回：晴天 25°C
  [工具调用] GetTime 被执行，返回：15:30:42
```

可以看到，ReduceAsync 仅被调用了 1 次，且是发生在对话流程的初始阶段。在后续的两次工具调用过程中，ReduceAsync 完全不会被触发。所有中间消息都会累积在 ChatHistoryProvider 中，直到整个对话流程结束。

### 对比总结

| 选项值 | ReduceAsync 调用次数（本演示） | 调用时机 |
|--------|------|------|
| true | 3 次 | 初始 + 每次工具调用完成后 |
| false（默认） | 1 次 | 仅在初始阶段 |

选择哪种设置取决于你的场景：

- 如果你的 Agent 可能会经历多轮工具调用、上下文容易膨胀，建议开启 RequirePerServiceCallChatHistoryPersistence 为 true，让 ReduceAsync 在每次服务调用后及时清理上下文。
- 如果你的 Agent 工具调用较少、对上下文完整性要求较高，保持默认的 false 可以避免过于频繁的裁剪。

## 扩展方法：RunStreamingAndLogToConsoleAsync

本演示还定义了一个简化的流式运行扩展方法，将 RunStreamingAsync 的 Text 增量输出到控制台：

```csharp
public static class AIAgentStreamingExtensions
{
    public static async Task RunStreamingAndLogToConsoleAsync(
        this AIAgent agent,
        IEnumerable<ChatMessage> messages,
        AgentSession? session = null,
        AgentRunOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        await foreach (var update in agent.RunStreamingAsync(messages, session, options, cancellationToken))
        {
            Console.Write(update.Text);
        }

        Console.WriteLine();
    }
}
```

由于本演示的核心关注点是 ReduceAsync 的调用时机而非流式输出的细节，因此这里略去了推理内容（Reasoning）的处理，只将文本内容直接打印到控制台。

## 关于实验性 API

在代码开头有一行 `#pragma warning disable MAAI001`，这是因为 ChatClientAgentOptions 中的部分 API 目前仍处于实验阶段，编译器会发出 MAAI001 警告。在演示代码中抑制此警告是安全的，但生产环境中请关注官方文档，确认相关 API 已经稳定后再使用。

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0ac919c6324e44add7cc01e53eb4b5046e324c3f/SemanticKernelSamples/FallnayyewelCeehowawcerjur) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0ac919c6324e44add7cc01e53eb4b5046e324c3f/SemanticKernelSamples/FallnayyewelCeehowawcerjur) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0ac919c6324e44add7cc01e53eb4b5046e324c3f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0ac919c6324e44add7cc01e53eb4b5046e324c3f
```

获取代码之后，进入 SemanticKernelSamples/FallnayyewelCeehowawcerjur 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

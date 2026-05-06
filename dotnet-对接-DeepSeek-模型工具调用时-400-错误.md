
# dotnet 对接 DeepSeek 模型工具调用时 400 错误

本文记录使用 Microsoft.Extensions.AI 对接 DeepSeek 模型，开启思考模式并使用工具调用时遇到的 reasoning_content 相关错误的解决方法，可直接配合 Microsoft Agent Framework 使用

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

最近我尝试用 OpenAI 官方 SDK 以兼容模式对接 DeepSeek 模型，开启深度思考模式之后，如果调用了自定义工具，就会在工具调用完成后发起下一轮请求的时候抛出 HTTP 400 错误，错误内容为 `The \`reasoning_content\` in the thinking mode must be passed back to the API`

翻查 DeepSeek 官方文档 <https://api-docs.deepseek.com/zh-cn/guides/thinking_mode#%E5%B7%A5%E5%85%B7%E8%B0%83%E7%94%A8> 可知，问题出在上下文回传的逻辑上：DeepSeek 的思考模式下，模型返回的 reasoning_content 思考内容，在后续的上下文请求里必须完整带回，包括工具调用完成后的续传请求。但是官方的 OpenAI .NET SDK 并没有适配 reasoning_content 这个扩展字段，在拼接上下文时会直接丢弃这部分内容，就触发了 API 的参数校验错误

为了解决这个问题，我采用了 [walterlv](https://blog.walterlv.com/) 封装的 DeepSeekChatClient 实现，它完全兼容 Microsoft.Extensions.AI 的 IChatClient 接口，原生支持了 DeepSeek 的思考模式和 reasoning_content 的上下文回传逻辑，完美解决了这个问题。我还专门将其封装为 Microsoft.Extensions.AI.DeepSeek 库，可以直接在项目中引入使用。

## 项目配置

Microsoft.Extensions.AI.DeepSeek 库支持 net6.0、net8.0 和 net10.0 框架，依赖 Microsoft.Extensions.AI 10.5.2 版本，项目的 csproj 配置如下：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net10.0;net6.0;net8.0</TargetFrameworks>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.AI" Version="10.5.2" />
  </ItemGroup>
</Project>
```

你只需要将这个项目添加到你的解决方案中，或者打包为 NuGet 包引入即可使用。

## 核心实现原理

DeepSeekChatClient 解决问题的核心逻辑是在构建请求消息的时候，会将上下文里的 TextReasoningContent 内容序列化为 reasoning_content 字段回传给 API，不会丢失思考内容。对应的实现代码如下：

```csharp
foreach (var content in message.Contents)
{
    switch (content)
    {
        case TextContent textContent when !string.IsNullOrEmpty(textContent.Text):
            textBuilder.Append(textContent.Text);
            break;
        case TextReasoningContent reasoningContent when !string.IsNullOrEmpty(reasoningContent.Text):
            // 收集思考内容，后续回传给 API
            reasoningBuilder.Append(reasoningContent.Text);
            break;
        case FunctionCallContent functionCall:
            toolCalls ??= [];
            toolCalls.Add(new JsonObject
            {
                ["id"] = functionCall.CallId,
                ["type"] = "function",
                ["function"] = new JsonObject
                {
                    ["name"] = functionCall.Name,
                    ["arguments"] = JsonSerializer.Serialize(functionCall.Arguments, SerializerOptions),
                },
            });
            break;
    }
}
// 构建请求时带上 reasoning_content 字段
yield return new JsonObject
{
    ["role"] = role,
    ["content"] = text,
    ["reasoning_content"] = reasoningBuilder.Length > 0 ? reasoningBuilder.ToString() : null,
    ["tool_calls"] = toolCalls,
};
```

## 整个代码

```csharp
namespace Microsoft.Extensions.AI.DeepSeek;

using Microsoft.Extensions.AI;

using System.Net.Http.Headers;
using System.Runtime.CompilerServices;
using System.Text;
using System.Text.Json;
using System.Text.Json.Nodes;
using System.Text.Json.Serialization;

public sealed partial class DeepSeekChatClient : IChatClient
{
    private static readonly JsonSerializerOptions SerializerOptions = new(JsonSerializerDefaults.Web)
    {
        DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
    };

    private readonly HttpClient _httpClient;
    private readonly bool _ownsHttpClient;
    private readonly Uri _baseUri;
    private readonly string _apiKey;
    private readonly string _defaultModelId;
    private readonly bool _enableThinkingMode;
    private readonly int _reasoningBudgetTokens;
    private readonly ChatClientMetadata _metadata;

    public DeepSeekChatClient
    (
        string apiKey,
        string modelId,
        string baseUrl = "https://api.deepseek.com/v1",
        bool enableThinkingMode = true,
        int reasoningBudgetTokens = 8000,
        HttpClient? httpClient = null
    )
    {
        if (string.IsNullOrWhiteSpace(apiKey))
        {
            throw new ArgumentException("Value cannot be null or whitespace.", nameof(apiKey));
        }

        if (string.IsNullOrWhiteSpace(modelId))
        {
            throw new ArgumentException("Value cannot be null or whitespace.", nameof(modelId));
        }

        if (string.IsNullOrWhiteSpace(baseUrl))
        {
            throw new ArgumentException("Value cannot be null or whitespace.", nameof(baseUrl));
        }

        _httpClient = httpClient ?? new HttpClient();
        _ownsHttpClient = httpClient is null;
        _baseUri = new Uri($"{baseUrl.AsSpan().TrimEnd('/')}/", UriKind.Absolute);
        _apiKey = apiKey;
        _defaultModelId = modelId;
        _enableThinkingMode = enableThinkingMode;
        _reasoningBudgetTokens = reasoningBudgetTokens;
        _metadata = new ChatClientMetadata("DeepSeek", _baseUri, modelId);
    }

    public async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        ArgumentNullException.ThrowIfNull(messages);

        List<AIContent> responseContents = [];
        UsageDetails? usage = null;
        ChatFinishReason? finishReason = null;
        string? responseId = null;
        string? modelId = null;
        DateTimeOffset? createdAt = null;
        object? rawRepresentation = null;
        ChatRole role = ChatRole.Assistant;

        await foreach (var update in GetStreamingResponseAsync(messages, options, cancellationToken))
        {
            if (update.Role is { } updateRole)
            {
                role = updateRole;
            }

            responseId ??= update.ResponseId;
            modelId ??= update.ModelId;
            createdAt ??= update.CreatedAt;
            finishReason = update.FinishReason ?? finishReason;
            rawRepresentation = update.RawRepresentation ?? rawRepresentation;

            foreach (var content in update.Contents)
            {
                if (content is UsageContent usageContent)
                {
                    usage = usageContent.Details;
                    continue;
                }

                responseContents.Add(content);
            }
        }

        var responseMessage = new ChatMessage(role, responseContents)
        {
            CreatedAt = createdAt,
            MessageId = responseId,
            RawRepresentation = rawRepresentation,
        };

        return new ChatResponse(responseMessage)
        {
            ResponseId = responseId,
            ModelId = modelId ?? options?.ModelId ?? _defaultModelId,
            CreatedAt = createdAt,
            FinishReason = finishReason,
            Usage = usage,
            RawRepresentation = rawRepresentation,
        };
    }

    public async IAsyncEnumerable<ChatResponseUpdate> GetStreamingResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        ArgumentNullException.ThrowIfNull(messages);

        var requestJson = BuildRequest(messages, options);
        using var request = new HttpRequestMessage(HttpMethod.Post, new Uri(_baseUri, "chat/completions"))
        {
            Content = new StringContent(requestJson.ToJsonString(SerializerOptions), Encoding.UTF8, "application/json"),
        };

        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", _apiKey);

        using var response = await _httpClient.SendAsync(request, HttpCompletionOption.ResponseHeadersRead, cancellationToken).ConfigureAwait(false);
        if (!response.IsSuccessStatusCode)
        {
            var errorBody = await response.Content.ReadAsStringAsync(cancellationToken).ConfigureAwait(false);
            throw new InvalidOperationException(MapErrorToMessage((int) response.StatusCode, errorBody));
        }

        using var stream = await response.Content.ReadAsStreamAsync(cancellationToken).ConfigureAwait(false);
        using var reader = new StreamReader(stream, Encoding.UTF8);

        string? responseId = null;
        string? modelId = null;
        DateTimeOffset? createdAt = null;
        ChatFinishReason? finishReason = null;
        UsageDetails? usage = null;
        var toolCallAccumulators = new Dictionary<int, ToolCallAccumulator>();

        await foreach (var chunk in ReadSseChunksAsync(reader, cancellationToken))
        {
            responseId ??= chunk.Id;
            modelId ??= chunk.Model;
            if (createdAt is null && chunk.Created is { } created)
            {
                createdAt = DateTimeOffset.FromUnixTimeSeconds(created);
            }

            var choice = chunk.Choices?.FirstOrDefault();
            if (choice?.Delta is { } delta)
            {
                List<AIContent> contents = [];

                if (!string.IsNullOrEmpty(delta.ReasoningContent))
                {
                    contents.Add(new TextReasoningContent(delta.ReasoningContent)
                    {
                        RawRepresentation = chunk,
                    });
                }

                if (!string.IsNullOrEmpty(delta.Content))
                {
                    contents.Add(new TextContent(delta.Content)
                    {
                        RawRepresentation = chunk,
                    });
                }

                if (delta.ToolCalls is { Length: > 0 })
                {
                    foreach (var toolCall in delta.ToolCalls)
                    {
                        if (!toolCallAccumulators.TryGetValue(toolCall.Index, out var accumulator))
                        {
                            accumulator = new ToolCallAccumulator();
                            toolCallAccumulators[toolCall.Index] = accumulator;
                        }

                        accumulator.Apply(toolCall);
                    }
                }

                if (contents.Count > 0)
                {
                    yield return new ChatResponseUpdate(ChatRole.Assistant, contents)
                    {
                        ResponseId = responseId,
                        ModelId = modelId,
                        CreatedAt = createdAt,
                        RawRepresentation = chunk,
                    };
                }
            }

            if (!string.IsNullOrWhiteSpace(choice?.FinishReason))
            {
                finishReason = ParseFinishReason(choice.FinishReason);
            }

            if (chunk.Usage is { } chunkUsage)
            {
                usage = CreateUsageDetails(chunkUsage);
            }
        }

        List<AIContent> finalContents = [];
        foreach (var toolCall in toolCallAccumulators.OrderBy(static pair => pair.Key).Select(static pair => pair.Value))
        {
            finalContents.Add(toolCall.ToFunctionCallContent());
        }

        if (usage is not null)
        {
            finalContents.Add(new UsageContent(usage));
        }

        if (finalContents.Count > 0 || finishReason is not null)
        {
            yield return new ChatResponseUpdate(ChatRole.Assistant, finalContents)
            {
                ResponseId = responseId,
                ModelId = modelId ?? options?.ModelId ?? _defaultModelId,
                CreatedAt = createdAt,
                FinishReason = finishReason,
            };
        }
    }

    public object? GetService(Type serviceType, object? serviceKey = null)
    {
        ArgumentNullException.ThrowIfNull(serviceType);

        if (serviceKey is not null)
        {
            return null;
        }

        if (serviceType.IsInstanceOfType(this))
        {
            return this;
        }

        if (serviceType.IsInstanceOfType(_metadata))
        {
            return _metadata;
        }

        if (serviceType.IsInstanceOfType(_httpClient))
        {
            return _httpClient;
        }

        return null;
    }

    public void Dispose()
    {
        if (_ownsHttpClient)
        {
            _httpClient.Dispose();
        }
    }

    private JsonObject BuildRequest(IEnumerable<ChatMessage> messages, ChatOptions? options)
    {
        var request = new JsonObject
        {
            ["model"] = options?.ModelId ?? _defaultModelId,
            ["stream"] = true,
            ["stream_options"] = new JsonObject { ["include_usage"] = true },
            ["messages"] = BuildMessages(messages, options),
        };

        if (options?.Temperature is { } temperature)
        {
            request["temperature"] = JsonValue.Create(temperature);
        }

        if (options?.TopP is { } topP)
        {
            request["top_p"] = JsonValue.Create(topP);
        }

        if (options?.MaxOutputTokens is { } maxOutputTokens)
        {
            request["max_tokens"] = JsonValue.Create(maxOutputTokens);
        }

        if (options?.StopSequences is { Count: > 0 } stopSequences)
        {
            var stop = new JsonArray();
            foreach (var stopSequence in stopSequences)
            {
                stop.Add(stopSequence);
            }

            request["stop"] = stop;
        }

        if (_enableThinkingMode)
        {
            request["thinking"] = new JsonObject
            {
                ["type"] = "enabled",
                ["budget_tokens"] = _reasoningBudgetTokens,
            };
        }

        var tools = BuildTools(options?.Tools);
        if (tools.Count > 0)
        {
            request["tools"] = tools;
            request["tool_choice"] = "auto";
        }

        return request;
    }

    private static JsonArray BuildMessages(IEnumerable<ChatMessage> messages, ChatOptions? options)
    {
        JsonArray jsonMessages = [];

        if (!string.IsNullOrWhiteSpace(options?.Instructions))
        {
            jsonMessages.Add(new JsonObject
            {
                ["role"] = "system",
                ["content"] = options.Instructions,
            });
        }

        foreach (var message in messages)
        {
            foreach (var jsonMessage in BuildMessages(message))
            {
                jsonMessages.Add(jsonMessage);
            }
        }

        return jsonMessages;
    }

    private static IEnumerable<JsonObject> BuildMessages(ChatMessage message)
    {
        ArgumentNullException.ThrowIfNull(message);

        if (message.Role == ChatRole.Tool)
        {
            foreach (var content in message.Contents)
            {
                if (content is FunctionResultContent functionResult)
                {
                    yield return new JsonObject
                    {
                        ["role"] = "tool",
                        ["tool_call_id"] = functionResult.CallId,
                        ["content"] = SerializeToolResult(functionResult.Result),
                    };
                }
            }

            yield break;
        }

        string? text = null;
        string? reasoning = null;
        JsonArray? toolCalls = null;

        if (message.Contents.Count > 0)
        {
            var textBuilder = new StringBuilder();
            var reasoningBuilder = new StringBuilder();

            foreach (var content in message.Contents)
            {
                switch (content)
                {
                    case TextContent textContent when !string.IsNullOrEmpty(textContent.Text):
                        textBuilder.Append(textContent.Text);
                        break;
                    case TextReasoningContent reasoningContent when !string.IsNullOrEmpty(reasoningContent.Text):
                        reasoningBuilder.Append(reasoningContent.Text);
                        break;
                    case FunctionCallContent functionCall:
                        toolCalls ??= [];
                        toolCalls.Add(new JsonObject
                        {
                            ["id"] = functionCall.CallId,
                            ["type"] = "function",
                            ["function"] = new JsonObject
                            {
                                ["name"] = functionCall.Name,
#if NET8_0_OR_GREATER
                                ["arguments"] = JsonSerializer.Serialize(functionCall.Arguments, SourceGenerationContext.Default.Options),
#else
                                ["arguments"] = JsonSerializer.Serialize(functionCall.Arguments, SerializerOptions),
#endif
                            },
                        });
                        break;
                }
            }

            text = textBuilder.Length > 0 ? textBuilder.ToString() : null;
            reasoning = reasoningBuilder.Length > 0 ? reasoningBuilder.ToString() : null;
        }
        else if (!string.IsNullOrWhiteSpace(message.Text))
        {
            text = message.Text;
        }

        var role = message.Role == ChatRole.System ? "system"
            : message.Role == ChatRole.Assistant ? "assistant"
            : "user";

        yield return new JsonObject
        {
            ["role"] = role,
            ["content"] = text,
            ["reasoning_content"] = reasoning,
            ["tool_calls"] = toolCalls,
        };
    }

    private static JsonArray BuildTools(IList<AITool>? tools)
    {
        JsonArray jsonTools = [];
        if (tools is null)
        {
            return jsonTools;
        }

        foreach (var tool in tools)
        {
            if (tool is not AIFunctionDeclaration function)
            {
                continue;
            }

            jsonTools.Add(new JsonObject
            {
                ["type"] = "function",
                ["function"] = new JsonObject
                {
                    ["name"] = function.Name,
                    ["description"] = function.Description,
                    ["parameters"] = ToJsonNode(function.JsonSchema) ?? new JsonObject(),
                },
            });
        }

        return jsonTools;
    }

    private static JsonNode? ToJsonNode(object? value)
    {
        return value switch
        {
            null => null,
            JsonNode node => node.DeepClone(),
            JsonElement element => JsonNode.Parse(element.GetRawText()),
            JsonDocument document => JsonNode.Parse(document.RootElement.GetRawText()),
            string text when !string.IsNullOrWhiteSpace(text) => JsonNode.Parse(text),
#if NET8_0_OR_GREATER
            _ => JsonSerializer.SerializeToNode(value, SourceGenerationContext.Default.Options),
#else
            _ => JsonSerializer.SerializeToNode(value, SerializerOptions),
#endif
        };
    }

    private static string SerializeToolResult(object? result)
    {
        return result switch
        {
            null => "null",
            string text => text,
            JsonElement element => element.GetRawText(),
            JsonDocument document => document.RootElement.GetRawText(),
#if NET8_0_OR_GREATER
            _ => JsonSerializer.Serialize(result, SourceGenerationContext.Default.Options),
#else
            _ => JsonSerializer.Serialize(result, SerializerOptions),
#endif
        };
    }

    private static async IAsyncEnumerable<DeepSeekChatChunk> ReadSseChunksAsync(
        StreamReader reader,
        [EnumeratorCancellation] CancellationToken cancellationToken)
    {
        while (true)
        {
#if NET8_0_OR_GREATER
            var line = await reader.ReadLineAsync(cancellationToken).ConfigureAwait(false);
#else
            var line = await reader.ReadLineAsync().ConfigureAwait(false);
#endif
            if (line is null)
            {
                yield break;
            }

            if (!line.StartsWith("data: ", StringComparison.Ordinal))
            {
                continue;
            }

            var payload = line["data: ".Length..];
            if (payload == "[DONE]")
            {
                yield break;
            }

            if (string.IsNullOrWhiteSpace(payload))
            {
                continue;
            }

            DeepSeekChatChunk? chunk;
            try
            {
#if NET8_0_OR_GREATER
                chunk = JsonSerializer.Deserialize(payload, SourceGenerationContext.Default.DeepSeekChatChunk);
#else
                chunk = JsonSerializer.Deserialize<DeepSeekChatChunk>(payload, SerializerOptions);
#endif
            }
            catch (JsonException)
            {
                continue;
            }

            if (chunk is not null)
            {
                yield return chunk;
            }
        }
    }

    private static UsageDetails CreateUsageDetails(DeepSeekUsage usage)
    {
        var usageDetails = new UsageDetails
        {
            InputTokenCount = usage.PromptTokens,
            OutputTokenCount = usage.CompletionTokens,
            TotalTokenCount = usage.TotalTokens > 0 ? usage.TotalTokens : usage.PromptTokens + usage.CompletionTokens,
            CachedInputTokenCount = usage.PromptCacheHitTokens,
            ReasoningTokenCount = usage.CompletionTokensDetails?.ReasoningTokens,
        };

        if (usage.PromptCacheMissTokens > 0)
        {
            usageDetails.AdditionalCounts = new AdditionalPropertiesDictionary<long>
            {
                ["prompt_cache_miss_tokens"] = usage.PromptCacheMissTokens,
            };
        }

        return usageDetails;
    }

    private static ChatFinishReason? ParseFinishReason(string? finishReason)
    {
        return finishReason switch
        {
            null => null,
            "stop" => ChatFinishReason.Stop,
            "length" => ChatFinishReason.Length,
            "tool_calls" => ChatFinishReason.ToolCalls,
            "content_filter" => ChatFinishReason.ContentFilter,
            _ => new ChatFinishReason(finishReason),
        };
    }

    private static string MapErrorToMessage(int statusCode, string responseBody)
    {
        string? errorCode = null;
        string? errorMessage = null;

        try
        {
#if NET8_0_OR_GREATER
            var error = JsonSerializer.Deserialize(responseBody, SourceGenerationContext.Default.DeepSeekErrorResponse);
#else
            var error = JsonSerializer.Deserialize<DeepSeekErrorResponse>(responseBody, SerializerOptions);
#endif
            errorCode = error?.Error?.Code;
            errorMessage = error?.Error?.Message;
        }
        catch (JsonException)
        {
        }

        var detail = errorCode switch
        {
            "insufficient_balance" => "账户余额不足,请充值后重试。",
            "rate_limit_exceeded" => "请求速率超限,请稍后重试。",
            "context_length_exceeded" => "输入内容超出模型上下文限制。",
            "invalid_api_key" => "API Key 无效,请检查配置。",
            "model_not_found" => "模型不存在,请检查模型名称。",
            "server_error" => "服务端内部错误,请稍后重试。",
            _ => null,
        };

        if (detail is not null)
        {
            return $"API 错误（HTTP {statusCode}, code={errorCode}）：{detail}";
        }

        if (!string.IsNullOrWhiteSpace(errorMessage))
        {
            return $"API 错误（HTTP {statusCode}）：{errorMessage}";
        }

        return $"API 请求失败,状态码 {statusCode}。";
    }

#if NET8_0_OR_GREATER
    [JsonSourceGenerationOptions(
        DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
        PropertyNamingPolicy = JsonKnownNamingPolicy.CamelCase)]
    [JsonSerializable(typeof(DeepSeekChatChunk))]
    [JsonSerializable(typeof(DeepSeekErrorResponse))]
    [JsonSerializable(typeof(Dictionary<string, JsonElement>))]
    [JsonSerializable(typeof(IDictionary<string, object>), GenerationMode = JsonSourceGenerationMode.Serialization)]
    private partial class SourceGenerationContext : JsonSerializerContext
    {
    }
#endif

    private sealed class ToolCallAccumulator
    {
        private string? _id;
        private string? _name;
        private readonly StringBuilder _arguments = new();

        public void Apply(DeepSeekToolCallDelta delta)
        {
            if (!string.IsNullOrWhiteSpace(delta.Id))
            {
                _id = delta.Id;
            }

            if (!string.IsNullOrWhiteSpace(delta.Function?.Name))
            {
                _name = delta.Function.Name;
            }

            if (!string.IsNullOrEmpty(delta.Function?.Arguments))
            {
                _arguments.Append(delta.Function.Arguments);
            }
        }

        public FunctionCallContent ToFunctionCallContent()
        {
            var argumentsJson = _arguments.Length > 0 ? _arguments.ToString() : "{}";
            return new FunctionCallContent(
                _id ?? Guid.NewGuid().ToString("N"),
                _name ?? string.Empty,
                ParseArguments(argumentsJson))
            {
                RawRepresentation = argumentsJson,
            };
        }

        private static Dictionary<string, object?> ParseArguments(string argumentsJson)
        {
            try
            {
#if NET8_0_OR_GREATER
                var json = JsonSerializer.Deserialize<Dictionary<string, JsonElement>>(argumentsJson, SourceGenerationContext.Default.Options);
#else
                var json = JsonSerializer.Deserialize<Dictionary<string, JsonElement>>(argumentsJson, SerializerOptions);
#endif
                if (json is null)
                {
                    return [];
                }

                return json.ToDictionary(static pair => pair.Key, static pair => (object?) pair.Value.Clone(), StringComparer.Ordinal);
            }
            catch (JsonException)
            {
                return [];
            }
        }
    }

    private sealed class DeepSeekChatChunk
    {
        [JsonPropertyName("id")]
        public string? Id { get; set; }

        [JsonPropertyName("model")]
        public string? Model { get; set; }

        [JsonPropertyName("created")]
        public long? Created { get; set; }

        [JsonPropertyName("choices")]
        public DeepSeekChoice[]? Choices { get; set; }

        [JsonPropertyName("usage")]
        public DeepSeekUsage? Usage { get; set; }
    }

    private sealed class DeepSeekChoice
    {
        [JsonPropertyName("delta")]
        public DeepSeekDelta? Delta { get; set; }

        [JsonPropertyName("finish_reason")]
        public string? FinishReason { get; set; }
    }

    private sealed class DeepSeekDelta
    {
        [JsonPropertyName("content")]
        public string? Content { get; set; }

        [JsonPropertyName("reasoning_content")]
        public string? ReasoningContent { get; set; }

        [JsonPropertyName("tool_calls")]
        public DeepSeekToolCallDelta[]? ToolCalls { get; set; }
    }

    private sealed class DeepSeekToolCallDelta
    {
        [JsonPropertyName("index")]
        public int Index { get; set; }

        [JsonPropertyName("id")]
        public string? Id { get; set; }

        [JsonPropertyName("function")]
        public DeepSeekFunctionDelta? Function { get; set; }
    }

    private sealed class DeepSeekFunctionDelta
    {
        [JsonPropertyName("name")]
        public string? Name { get; set; }

        [JsonPropertyName("arguments")]
        public string? Arguments { get; set; }
    }

    private sealed class DeepSeekUsage
    {
        [JsonPropertyName("prompt_tokens")]
        public int PromptTokens { get; set; }

        [JsonPropertyName("completion_tokens")]
        public int CompletionTokens { get; set; }

        [JsonPropertyName("total_tokens")]
        public int TotalTokens { get; set; }

        [JsonPropertyName("prompt_cache_hit_tokens")]
        public int PromptCacheHitTokens { get; set; }

        [JsonPropertyName("prompt_cache_miss_tokens")]
        public int PromptCacheMissTokens { get; set; }

        [JsonPropertyName("completion_tokens_details")]
        public DeepSeekCompletionTokensDetails? CompletionTokensDetails { get; set; }
    }

    private sealed class DeepSeekCompletionTokensDetails
    {
        [JsonPropertyName("reasoning_tokens")]
        public int? ReasoningTokens { get; set; }
    }

    private sealed class DeepSeekErrorResponse
    {
        [JsonPropertyName("error")]
        public DeepSeekError? Error { get; set; }
    }

    private sealed class DeepSeekError
    {
        [JsonPropertyName("code")]
        public string? Code { get; set; }

        [JsonPropertyName("message")]
        public string? Message { get; set; }
    }
}
```

同时这个实现还完整支持 DeepSeek 的所有扩展特性：流式响应输出、思考内容流式返回、工具调用完整流程、错误码友好转换、推理Token消耗统计等，甚至会把缓存命中、推理Token消耗这些扩展字段都映射到 UsageDetails 里，方便业务统计成本。

## 使用示例

初始化 DeepSeekChatClient 的方式非常简单，只需要传入你的 DeepSeek API Key 和要使用的模型ID即可，也可以根据业务需要配置思考模式开关、推理预算Token等参数。以下是完整的工具调用测试代码：

```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;
using Microsoft.Extensions.AI.DeepSeek;
using OpenAI;
using System.ClientModel;

// 读取你的 DeepSeek API Key，记得替换为你自己的 Key 文件路径
var keyFile = @"C:\lindexi\Work\deepseek.txt";
var key = File.ReadAllText(keyFile);

// 初始化 DeepSeekChatClient，默认开启思考模式
using var chatClient = new DeepSeekChatClient(key, "deepseek-v4-pro");

// 如果使用 OpenAIClient 对接，就会出现本文提到的错误
//var openAiClient = new OpenAIClient(new ApiKeyCredential(key), new OpenAIClientOptions()
//{
//    Endpoint = new Uri("https://api.deepseek.com/v1")
//});
//var openAiChatClient = openAiClient.GetChatClient("deepseek-v4-pro").AsIChatClient();

// 构建带工具调用的 AI Agent，这里注册了查询天气的工具
ChatClientAgent agent = chatClient
    .AsBuilder()
    .BuildAIAgent(tools: [AIFunctionFactory.Create(GetWeather)]);

var session = await agent.CreateSessionAsync();
bool isThinking = false;

// 流式获取响应内容，区分输出思考内容和最终回答
await foreach (var agentResponseUpdate in agent.RunStreamingAsync("北京天气怎样", session))
{
    foreach (var aiContent in agentResponseUpdate.Contents)
    {
        if (aiContent is TextReasoningContent textReasoningContent)
        {
            if (string.IsNullOrEmpty(textReasoningContent.Text))
            {
                continue;
            }
            isThinking = true;
            Console.Write(textReasoningContent.Text);
        }
        else if (aiContent is TextContent textContent)
        {
            if (string.IsNullOrEmpty(textContent.Text))
            {
                continue;
            }
            if (isThinking)
            {
                Console.WriteLine();
            }
            isThinking = false;
            Console.Write(textContent.Text);
        }
    }
}

Console.WriteLine();
Console.WriteLine("Hello, World!");

// 自定义的天气查询工具
string GetWeather(string city)
{
    return $"The weather in {city} is sunny.";
}
```

运行以上代码即可看到模型先输出思考过程，然后调用天气工具，最后返回最终回答，全程不会再出现 reasoning_content 相关的错误。

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bc29ceb1611cb60c264143c3b13e408346ae14e3/SemanticKernelSamples/Microsoft.Agents.AI.Extensions) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/bc29ceb1611cb60c264143c3b13e408346ae14e3/SemanticKernelSamples/Microsoft.Agents.AI.Extensions) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bc29ceb1611cb60c264143c3b13e408346ae14e3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bc29ceb1611cb60c264143c3b13e408346ae14e3
```

获取代码之后，进入 SemanticKernelSamples/Microsoft.Agents.AI.Extensions 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

## 其他异常情况

由于 DeepSeek 对工具要求比较严格，如工具名只能符合 `^[a-zA-Z0-9_-]+$` 正则条件。如果传入不符合正则约束的工具名，将会遇到 HTTP 400 错误：

```
HTTP 400：Invalid 'tools[0].function.name': string does not match pattern. Expected a string that matches the pattern '^[a-zA-Z0-9_-]+$'.”
```

解决方法为修改工具名，让工具名符合规范




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
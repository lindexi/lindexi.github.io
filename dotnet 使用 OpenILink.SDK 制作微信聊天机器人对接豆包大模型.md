# dotnet 使用 OpenILink.SDK 制作微信聊天机器人对接豆包大模型

本文记录如何使用第三方开源的 OpenILink.SDK 快速搭建微信聊天机器人，并且对接字节跳动豆包大模型实现智能回复功能，本文提供的代码可以直接在实际项目中复用

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

最近需要做一个微信端的智能聊天机器人，找到了第三方开源的 OpenILink.SDK 项目，整体用下来体验还不错，先给大家介绍一下这个SDK的基础信息：

- 开源地址：<https://github.com/openilink/openilink-sdk-csharp/>
- 支持的框架版本：`net462`、`netstandard2.0`、`net8.0`，同一个NuGet包可以同时兼容老的 `.NET Framework`、`.NET Core` 和现代 .NET 版本，跨版本支持做得很好
- 我拉了代码下来看，虽然AI贡献的代码占比不低，但整体质量还算过关，我个人评价是优于「树上的小猫咪」的同类型项目质量

## 环境准备

首先我们需要安装两个NuGet包，一个是OpenILink.SDK用来对接微信，另一个是Microsoft.Agents.AI.OpenAI用来对接豆包大模型

编辑你的csproj文件，添加如下引用：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="OpenILink.SDK" Version="1.0.0" />
    <PackageReference Include="Microsoft.Agents.AI.OpenAI" Version="1.0.0" />
  </ItemGroup>
</Project>
```

你也可以根据自己的项目需要修改TargetFramework，最低可以支持到.NET Framework 4.6.2。

## OpenILink.SDK 基础使用

### 创建客户端

OpenILink.SDK提供了多种创建客户端的方式，最基础的写法是直接传入已经有的Token：

```csharp
using var client = OpenILinkClient.Create(token);
```

如果需要自定义配置，比如修改请求地址、超时时间等，可以使用Builder模式或者OpenILinkClientOptions：

```csharp
// Builder模式
using var httpClient = new HttpClient();
using var client = OpenILinkClient.Builder()
    .Token(token)
    .BaseUri("https://ilinkai.weixin.qq.com/")
    .ApiTimeout(TimeSpan.FromSeconds(15))
    .Build();
// 配置类模式
var options = new OpenILinkClientOptions(token)
{
    BaseUri = new Uri("https://ilinkai.weixin.qq.com/"),
    LoginTimeout = TimeSpan.FromMinutes(8)
};
using var client = new OpenILinkClient(options);
```

### 登录

首次运行的时候没有登录Token，我们可以使用扫码登录的方式，登录成功之后SDK会返回Token，我们可以把它存在本地，下次运行直接读取就不用再扫码了：

```csharp
var login = await client.LoginWithQrAsync(ShowQrCode, OnScanned, OnExpired);
if (login.Connected)
{
    // 保存Token到本地，下次启动直接使用
    File.WriteAllText("bot_token.txt", login.BotToken ?? string.Empty);
}
// 显示二维码，这里拿到的是一个网页地址，里面用Canvas绘制二维码，我这里直接调用系统浏览器打开
void ShowQrCode(string qrCodeImageUrl)
{
    Process.Start(new ProcessStartInfo(qrCodeImageUrl) { UseShellExecute = true });
}
void OnScanned() => Console.WriteLine("已扫码，请在微信端确认登录");
void OnExpired(int attempt, int maxAttempt) => Console.WriteLine($"二维码过期，正在刷新 ({attempt}/{maxAttempt})");
```

### 接收和回复消息

调用MonitorAsync方法就可以开始监听消息，SDK内部已经自动处理了重试、退避、消息位置记录等逻辑，我们只需要处理收到的消息即可：

```csharp
await client.MonitorAsync(HandleMessageAsync, new MonitorOptions
{
    // 上次保存的消息位置Buffer，避免重启后收到大量历史消息
    InitialBuffer = File.Exists("get_updates_buf.txt") ? File.ReadAllText("get_updates_buf.txt") : string.Empty,
    // Buffer更新时保存到本地
    OnBufferUpdated = buffer => File.WriteAllText("get_updates_buf.txt", buffer),
    OnError = exception => Console.Error.WriteLine($"出错：{exception.Message}"),
    OnSessionExpired = () => Console.Error.WriteLine("会话过期，请重新登录")
});
// 消息处理逻辑
Task HandleMessageAsync(WeixinMessage message)
{
    var text = message.ExtractText();
    if (string.IsNullOrWhiteSpace(text)) return Task.CompletedTask;
    // 直接回复消息
    return client.ReplyTextAsync(message, $"收到你的消息：{text}");
}
```

这里要特别注意`get_updates_buf.txt`的作用：它用来记录当前已经拉取到的最新消息的位置，如果没有这个文件，SDK启动时会拉取最近一段时间的所有未读消息，可能会导致重复处理历史消息，建议每次启动都读取保存的Buffer内容。

## 对接豆包大模型

接下来我们就可以把收到的用户消息转发给豆包大模型，拿到回复之后再返回给微信用户，完整的代码如下：

```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;
using OpenAI;
using OpenILink.SDK;
using System.ClientModel;
using System.Diagnostics;
// 初始化豆包客户端，请替换成你自己的密钥和模型端点
var keyFile = @"C:\lindexi\Work\Doubao.txt";
var key = File.ReadAllText(keyFile);
var openAiClient = new OpenAIClient(new ApiKeyCredential(key), new OpenAIClientOptions()
{
    Endpoint = new Uri("https://ark.cn-beijing.volces.com/api/v3"),
});
var chatClient = openAiClient.GetChatClient("ep-20260306101224-c8mtg"); // 替换成你自己的豆包模型ID
var agent = chatClient.AsIChatClient()
    .AsBuilder()
    .BuildAIAgent(new ChatClientAgentOptions()
    {
        ChatOptions = new ChatOptions()
        {
            // 可以在这里添加自定义工具，让豆包支持调用本地能力
            Tools = []
        }
    });
// 读取本地保存的Token
var tokenFilePath = Path.Join(AppContext.BaseDirectory, "Token.txt");
string? initialToken = File.Exists(tokenFilePath) ? File.ReadAllText(tokenFilePath) : null;
// 读取本地保存的消息Buffer
var bufferPath = Path.Join(AppContext.BaseDirectory, "GetUpdatesBuffer.txt");
string? initBuffer = File.Exists(bufferPath) ? File.ReadAllText(bufferPath) : null;
// 创建OpenILink客户端
var client = OpenILinkClient.Create(initialToken);
// 没有Token就扫码登录
if (string.IsNullOrWhiteSpace(client.Token))
{
    var login = await client.LoginWithQrAsync(ShowQrCode, OnScanned);
    if (!login.Connected)
    {
        Console.Error.WriteLine($"登录失败: {login.Message}");
        return;
    }
    File.WriteAllText(tokenFilePath, client.Token);
}
// 开始监听消息
await client.MonitorAsync(HandleMessageAsync, new MonitorOptions
{
    InitialBuffer = initBuffer,
    OnBufferUpdated = SaveBuffer,
    OnError = ReportError,
    OnSessionExpired = ReportSessionExpired
});

// 显示登录二维码
void ShowQrCode(string qrCodeImageDownloadUrl)
{
    Process.Start(new ProcessStartInfo(qrCodeImageDownloadUrl)
    {
        UseShellExecute = true
    });
}

void OnScanned() => Console.WriteLine("已扫码，请在微信端确认。");

// 处理收到的消息
async Task HandleMessageAsync(WeixinMessage message)
{
    var getConfigResponse = await client.GetConfigAsync(message.FromUserId, message.ContextToken!);
    var text = message.ExtractText();
    if (string.IsNullOrWhiteSpace(text)) return;
    // 发送正在输入的状态，提升用户体验
    await client.SendTypingAsync(message.FromUserId, getConfigResponse.TypingTicket!, TypingStatus.Typing);
    Console.WriteLine($"[{message.FromUserId}] {text}");
    // 调用豆包获取回复
    var agentResponse = await agent.RunAsync
    (
        [
            new ChatMessage(ChatRole.System,"你是一个充满积极向上情绪的聊天机器人"),
            new ChatMessage(ChatRole.User, text)
        ]
    );
    // 拼接回复内容，包含思考过程和Token消耗
    var reason = string.Empty;
    foreach (ChatMessage agentResponseMessage in agentResponse.Messages)
    {
        foreach (var textReasoningContent in agentResponseMessage.Contents.OfType<TextReasoningContent>())
        {
            reason += textReasoningContent.Text;
        }
    }
    var responseText = string.Empty;
    if (!string.IsNullOrEmpty(reason))
    {
        responseText =
            $"""
             思考：
             {reason.Trim()}
             -----------
             {agentResponse.Text}
             """;
    }
    // 追加Token消耗统计
    if (agentResponse.Usage is { } usage)
    {
        var usageText = $"本次对话总Token消耗：{usage.TotalTokenCount};输入Token消耗：{usage.InputTokenCount};输出Token消耗：{usage.OutputTokenCount},其中思考占{usage.ReasoningTokenCount??0}";
        responseText += $"\r\n-----------\r\n{usageText}";
    }
    Console.WriteLine($"[Bot] {responseText}");
    // 回复给微信用户
    await client.ReplyTextAsync(message, responseText);
}
void SaveBuffer(string buffer) => File.WriteAllText(bufferPath, buffer);
void ReportError(Exception exception) => Console.Error.WriteLine(exception.Message);
void ReportSessionExpired() => Console.Error.WriteLine("会话过期，请重新登录。");
```

## 注意事项

1. 请记得替换代码中的豆包API密钥、模型ID为你自己在火山引擎方舟平台申请的内容，密钥不要泄露到公开环境
2. 首次运行会自动打开二维码页面，使用你要作为机器人的微信号扫码登录即可，登录成功后Token会自动保存到本地，下次运行不需要再扫码
3. 如果不需要显示豆包的思考过程，只需要删掉拼接思考内容的代码，直接返回`agentResponse.Text`即可
4. SDK默认已经处理了多线程安全、消息去重、重试等逻辑，不需要自己额外实现
5. 如果需要发送图片、文件等媒体内容，可以使用SDK提供的`SendMediaFileAsync`、`SendImageAsync`等方法，具体可以看官方文档

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e7ac4f21c3fb7ea538d1e442e32479a0a3a8162b/SemanticKernelSamples/OpenILinkDemo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e7ac4f21c3fb7ea538d1e442e32479a0a3a8162b/SemanticKernelSamples/OpenILinkDemo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e7ac4f21c3fb7ea538d1e442e32479a0a3a8162b
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e7ac4f21c3fb7ea538d1e442e32479a0a3a8162b
```

获取代码之后，进入 SemanticKernelSamples/OpenILinkDemo 文件夹，即可获取到源代码

## 相关博客

- [C# Microsoft Agent Framework 与 豆包 对接](https://blog.lindexi.com/post/C-Microsoft-Agent-Framework-%E4%B8%8E-%E8%B1%86%E5%8C%85-%E5%AF%B9%E6%8E%A5.html )
<!-- [C# Microsoft Agent Framework 与 豆包 对接 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19494917 ) -->

- [Microsoft Agent Framework 取出 DeepSeek 思考内容](https://blog.lindexi.com/post/Microsoft-Agent-Framework-%E5%8F%96%E5%87%BA-DeepSeek-%E6%80%9D%E8%80%83%E5%86%85%E5%AE%B9.html )
<!-- [Microsoft Agent Framework 取出 DeepSeek 思考内容 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19635618 ) -->
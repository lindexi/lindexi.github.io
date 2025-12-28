---
title: Microsoft Agent Framework 与 DeepSeek 对接
description: 本文将告诉大家如何将 Microsoft Agent Framework 与 DeepSeek 对接
tags: 
category: 
---

<!-- 发布 -->
<!-- 博客 -->

## 准备工作

先使用手机号在 <https://platform.deepseek.com> 上注册账号

最后进入充值页面充值。如果没有充值，则后续 API 调用会返回 402 错误

最后进入 <https://platform.deepseek.com/api_keys> 创建 API key 且复制出来，后续步骤将会用到

## 安装库

按照 .NET 的惯例，使用前先使用 NuGet 安装对应的库

- Microsoft.Agents.AI.OpenAI

安装之后的 csproj 项目文件内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Agents.AI.OpenAI" Version="1.0.0-preview.251219.1" />
  </ItemGroup>

</Project>
```

## 编写代码

根据 DeepSeek 的官方文档，可以知道 DeepSeek 是能够完全兼容 OpenAI 的接口的，详细请看 <https://api-docs.deepseek.com/zh-cn/>

通过文档可以知道，只需要访问的是 <https://api.deepseek.com/v1> 链接，即可兼容 OpenAI 的接口。注，这里的 v1 不代表模型的版本，而是代表 API 的版本

通过与 OpenAI 的接口完全兼容的 <https://api.deepseek.com/v1> 链接，即可直接与 OpenAIClient 类型对接

对接代码如下

```csharp
var openAiClient = new OpenAIClient(new ApiKeyCredential(key), new OpenAIClientOptions()
{
    Endpoint = new Uri("https://api.deepseek.com/v1")
});
```

以上代码的 key 就是前面步骤保存的 API key 内容。为了安全起见，不应该将 API key 写入到代码里面，我就简单地放到 `C:\lindexi\Work\deepseek.txt` 文件里，随后用 `File.ReadAllText` 进行读取，其代码如下

```csharp
var keyFile = @"C:\lindexi\Work\deepseek.txt";
var key = File.ReadAllText(keyFile);

var openAiClient = new OpenAIClient(new ApiKeyCredential(key), new OpenAIClientOptions()
{
    Endpoint = new Uri("https://api.deepseek.com/v1")
});
```

根据 <https://api-docs.deepseek.com/zh-cn/quick_start/pricing> 文档可以知道，现在 DeepSeek 提供了这些模型可以被调用：

- deepseek-chat: DeepSeek-V3.2 （非思考模式）
- deepseek-reasoner: DeepSeek-V3.2（思考模式）

咱这里就随意选了 deepseek-chat 模型，代码如下

```csharp
var chatClient = openAiClient.GetChatClient("deepseek-chat");
```

拿到 ChatClient 之后，就可以通过 CreateAIAgent 扩展方法创建 ChatClientAgent 对象了，代码如下

```csharp
ChatClient chatClient = openAiClient.GetChatClient("deepseek-chat");

ChatClientAgent aiAgent = chatClient.CreateAIAgent();
```

现在拿到的 ChatClientAgent 对象就可以直接开始跑了，如下面的测试例子

```csharp
AgentRunResponse runResponse = await aiAgent.RunAsync("告诉我一个关于海盗的笑话");
Console.WriteLine(runResponse);
```

我跑了一下，可以在控制台看到如下输出信息，看起来 DeepSeek 的笑话还是很好笑的

```
好的，这里有一个经典的海盗笑话：

---

一个海盗走进酒吧，酒保看到他右眼戴着眼罩，左手是铁钩，右腿是木棍做的假肢，就问：“哇，你这身行头真厉害！是怎么弄成这样的？”

海盗说：“啊，这眼罩是因为有一天海鸥屎掉进我眼睛，我就瞎了。”

酒保一愣：“海鸥屎能让人瞎？”

海盗：“呃……其实是我刚戴上眼罩第一天，不太习惯，用铁钩手揉眼睛时戳瞎的。”
```

如果发现在调用的时候提示的 HTTP 错误是 402 则证明账户上已经没钱了，或还没进行充值。详细错误信息请参阅 <https://api-docs.deepseek.com/zh-cn/quick_start/error_codes>

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/cb19960c943988735924144830886c76f20adae9/SemanticKernelSamples/BewukobuheQinalaykodall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/cb19960c943988735924144830886c76f20adae9/SemanticKernelSamples/BewukobuheQinalaykodall) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cb19960c943988735924144830886c76f20adae9
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cb19960c943988735924144830886c76f20adae9
```

获取代码之后，进入 SemanticKernelSamples/BewukobuheQinalaykodall 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

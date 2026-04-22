# 通过 OllamaSharp 对接 Microsoft Agent Framework 的方法

本文记录如何使用 OllamaSharp 对接 Microsoft.Agents 框架，调用本地部署的阿里千问 qwen3-vl:8b 多模态大模型，实现本地图片的内容识别能力，代码可直接集成到现有 .NET 项目中使用

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

## 前置准备

在开始之前，请确保你已经通过 Ollama 拉取了 `qwen3-vl:8b` 模型，且 Ollama 服务已经正常启动，可正常访问调用。

## 项目配置

首先创建一个 .NET 10 控制台项目，编辑 csproj 项目文件，添加对应的 NuGet 包引用，完整项目文件内容如下：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Agents.AI" Version="1.2.0" />
    <PackageReference Include="OllamaSharp" Version="5.4.25" />
  </ItemGroup>

</Project>
```

## 核心实现代码

完成项目配置之后即可编写核心调用逻辑，完整可运行代码如下：

```csharp
var ollamaEndpoint = new Uri("http://172.20.113.28:11434");
const string modelId = "qwen3-vl:8b";
const string imagePath = @"f:\temp\20260422085613.png";

if (!File.Exists(imagePath))
{
    throw new FileNotFoundException($"找不到测试图片：{imagePath}", imagePath);
}

var prompt = """
             Inspect this screenshot and report the visible work in 1-2 sentences.
             State which application, webpage, or document is on screen, what action is being performed, and any trustworthy details such as file names, channels, tabs, or subjects.
             Keep the description concrete, concise, and strictly based on what can actually be seen.

             GOOD EXAMPLES:
             ✓ "Visual Studio is open on `Program.cs`, building an Ollama image-analysis sample with Semantic Kernel packages."
             ✓ "A Gmail draft is open for a client update about the release schedule, with the compose window in focus."
             ✓ "A Slack thread in `#engineering` is discussing API throttling and retry behavior."

             BAD EXAMPLES:
             ✗ "Someone is working" (too generic)
             ✗ "A website is open" (does not identify the page)
             ✗ "The user is doing computer stuff" (not factual enough)
             """;

var imageBytes = await File.ReadAllBytesAsync(imagePath);
var mimeType = GetImageMimeType(imagePath);

var ollamaApiClient = new OllamaApiClient(ollamaEndpoint, modelId);
AIAgent agent = new ChatClientAgent
(
    ollamaApiClient,
    instructions:
    """
    You analyze desktop screenshots.
    Reply in Chinese with 1-2 sentences.
    Name the visible application, page, or document whenever it can be identified.
    Describe the current task and include only details that are actually visible.
    If a detail is unclear, say so instead of guessing.
    """
    );

var message = new ChatMessage
{
    Role = ChatRole.User,
    Contents = 
    [
        new TextContent(prompt),
        new DataContent(imageBytes, mimeType)
    ]
};

var agentResponse = await agent.RunAsync(message);

Console.WriteLine("识别结果：");
Console.WriteLine(agentResponse.Text);

return;

static string GetImageMimeType(string imagePath)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(imagePath);

    return Path.GetExtension(imagePath).ToLowerInvariant() switch
    {
        ".png" => "image/png",
        ".jpg" or ".jpeg" => "image/jpeg",
        ".webp" => "image/webp",
        ".gif" => "image/gif",
        ".bmp" => "image/bmp",
        _ => "application/octet-stream"
    };
}
```

代码核心逻辑说明：

1. 首先配置 Ollama 服务地址、模型ID、待识别图片路径，前置校验图片是否存在
2. 编写规范的提示词约束模型输出格式，避免返回过于笼统、不符合要求的结果
3. 读取图片二进制内容，通过 `GetImageMimeType` 方法匹配图片对应的MIME类型，避免模型无法识别图片格式
4. 初始化 Ollama 客户端和 Agent，指定系统提示词要求返回中文的简洁识别结果，不猜测不确定内容
5. 组装包含文本提示和图片内容的用户消息，调用 Agent 异步获取识别结果并输出到控制台

## 注意事项

- 请将代码中的 `ollamaEndpoint` 替换为你自己的 Ollama 服务地址，本地部署默认地址为 `http://localhost:11434`
- `modelId` 请替换为你实际在 Ollama 中拉取的千问多模态模型ID
- `imagePath` 请替换为你自己的测试图片路径
- 可根据业务需求调整提示词内容，比如要求模型识别特定类型的内容、输出指定格式的结果

运行项目即可在控制台看到对应图片的识别结果，复杂识别场景可尝试切换更大参数的千问多模态模型版本获得更优效果。

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c37c7909d355eb482f5a6b32db65162f91b53933/SemanticKernelSamples/KealeljelnawbeHeyuhearnalyo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/c37c7909d355eb482f5a6b32db65162f91b53933/SemanticKernelSamples/KealeljelnawbeHeyuhearnalyo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c37c7909d355eb482f5a6b32db65162f91b53933
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c37c7909d355eb482f5a6b32db65162f91b53933
```

获取代码之后，进入 SemanticKernelSamples/KealeljelnawbeHeyuhearnalyo 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )



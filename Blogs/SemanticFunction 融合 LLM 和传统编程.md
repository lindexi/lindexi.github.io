本文将继续和大家介绍 SemanticKernel 神奇的魔法，将使用 LLM 大语言模型编写的自然语言函数和传统的编程语言编写的函数融合到一起的例子。通过本文的例子，大家可以看到 SemanticKernel 框架所推荐的一个工作模式，同时可以更好的理解 SemanticKernel 框架的用法

<!--more-->


<!-- CreateTime:2023/9/4 8:45:17 -->
<!-- 标题： SemanticFunction 融合 LLM 和传统编程 -->
<!-- 发布 -->
<!-- 博客 -->

本文属于 SemanticKernel 入门系列博客，更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 或 [博客园的合集](https://www.cnblogs.com/lindexi/collections/6439)

开始之前先来聊聊本文的背景，本文将给出一个使用 SemanticKernel 框架连接魔法的 LLM 和传统的编程语言编写的函数的应用例子。这个例子所解决的问题是：我拿到了多个不同的数据集，我需要编写代码将数据集里面所提到的日期提取出来

更具体一点的实现是我拿到的多个不同的数据集里面，每个数据集对于日期的定义可能是不相同的格式，如以下的两个数据例子。如果不依靠 LLM 魔法的话，那我只能写多个正则表达式去适配多个不同的数据集了。但在本文的设定里面，我是不擅长写正则表达式的

```
var data1 =
"""
在2023年9月1号开始上课
在2023年9月2号开始准备教材
在2023年9月3号完成作业
""";

var data2 =
"""
在9.1.2023开始上课
在9.2.2023开始准备教材
在9.3.2023完成作业
""";
```

那如果全部都给 LLM 处理呢？理论上是可以的，但是存在两个问题，第一个是 LLM 可能不够擅长做比较大的数据集的处理，无论是成本方面还是本身 Token 长度限制方面。第二个就是我选用的 GPT 3.5 模型本身难以完成这项任务，当前的执行效率也不够高，需要跑半天才能完成，且即使完成之后后续对接解析结果也需要额外的工作量

那是否有比较完美的方案，同时规避了传统编程函数和 LLM 的缺点，且拥有两者的优势？这是当然有的，通过 SemanticKernel 框架的辅助下，咱可以各取两者的优点，让传统编程难以完成的事情交给 LLM 去实现，将 LLM 不擅长做的交给传统编程语言去完成

总的技术实现是，先对每个数据集分别进行处理。编写传统 C# 函数，取出数据集里面的代表数据，也就是第一行的字符串。接着将取出的代表数据给到使用自然语言编程函数的 GPT 进行处理，让 GPT 给出正则表达式字符串。再根据 GPT 给出的正则表达式字符串，传入到 C# 的正则类里面，让 C# 代码高效稳定处理数据集

如此实现既可以让开发者不用编写复杂的正则表达式，同时也可以使用一套代码处理多个不同的数据格式的数据集

接下来让大家看看 SemanticKernel 将 LLM 自然语言函数和传统编程融合到一起的威力

在开始编写代码之前，期望大家已经对 SemanticKernel 和 C# 语言有了入门的了解

新建一个 .NET 7 的控制台应用，编辑 csproj 项目文件，按照 dotnet 的习俗安装好各个 NuGet 库，修改之后的 csproj 项目文件代码大概如下。放心，在本文末尾将给出本文所有代码的下载方法

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="7.0.0" />
    <PackageReference Include="Microsoft.SemanticKernel" Version="0.20.230821.4-preview" />
  </ItemGroup>

</Project>
```

接着进行配置和初始化，本文这里用到了 AzureAI 的 GPT 3.5 模型，需要提前申请好。申请地址：https://aka.ms/oai/access

```csharp
using System.Text.RegularExpressions;

using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Orchestration;
using Microsoft.SemanticKernel.SkillDefinition;

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
    .Build();
```

关于以上配置，详细请参阅 [我的 SemanticKernel 合集博客](https://www.cnblogs.com/lindexi/collections/6439)

按照本文的技术实现设计，先编写 C# 函数，这里需要有两个函数，分别是取出数据集的第一行作为代表数据，以及使用 GPT 给到的正则表达式字符串进行处理数据集。根据 [SemanticKernel 自定义技能](https://blog.lindexi.com/post/dotnet-SemanticKernel-%E5%85%A5%E9%97%A8-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%8F%98%E9%87%8F%E5%92%8C%E6%8A%80%E8%83%BD.html ) 博客提供的方法，以下直接给出这两个 C# 函数编写的 TextSkill 技能代码

```csharp
class TextSkill
{
    [SKFunction]
    public string TakeFirstLine(string input, SKContext context)
    {
        context.Variables["text"] = input;
        var stringReader = new StringReader(input);
        return stringReader.ReadLine() ?? string.Empty;
    }

    [SKFunction]
    public void RegexMatchText(string input, string text, SKContext context)
    {
        foreach (Match match in Regex.Matches(text,input))
        {
            Console.WriteLine(match.Value);
        }
    }
}
```

上面代码里面的 TakeFirstLine 就是取出数据集的第一行作为代表数据，其中一个细节就是将输入的内容放入到 `text` 变量里面，而不是丢掉。放入到变量里面就可以方便让后续的 RegexMatchText 函数使用

在 RegexMatchText 就是根据 GPT 给出的正则表达式字符串，也就是 input 变量对应的值，对 TakeFirstLine 放入的 text 变量，也就是原始数据集进行处理。处理之后直接打印

完成 C# 函数编写之后，将 TextSkill 技能导入到 SemanticKernel 里

```csharp
kernel.ImportSkill(new TextSkill());
```

接下来请出百万炼丹师进行编写自然语言函数，让 GPT 可以可以从代表数据里面输出提取日期的正则表达式

```
const string FunctionDefinition =
"""
我有这样一段文本：
{ {$input} }
请你写一个正则表达式字符串，用来提取出日期

正则表达式字符串:
""";

kernel.CreateSemanticFunction(FunctionDefinition, maxTokens: 200, temperature: 0.5, functionName: "BuildRegexText");
```

调用 CreateSemanticFunction 时传入函数名，则可以自动注册到 SemanticKernel 框架里。现在咱拥有了三个函数，分别是两个 C# 代码编写的 TakeFirstLine 和 RegexMatchText 函数，以及使用自然语言编写的 BuildRegexText 魔法函数

按照 [SemanticKernel 管道式调用函数](https://blog.lindexi.com/post/dotnet-SemanticKernel-%E5%85%A5%E9%97%A8-%E5%B0%86%E6%8A%80%E8%83%BD%E5%AF%BC%E5%85%A5%E6%A1%86%E6%9E%B6.html ) 博客提供的方法，咱可以使用管道将以上几个函数排列组合放入到 SemanticKernel 执行

为了方便调用，这里编写了一个 C# 内部方法，方法的入参就是数据集

```csharp
async Task RunAsync(string data)
{
    await kernel.RunAsync
    (
        data,
        kernel.Skills.GetFunction("TakeFirstLine"),
        kernel.Skills.GetFunction("BuildRegexText"),
        kernel.Skills.GetFunction("RegexMatchText")
    );
}
```

以上代码就是按照本文开始的设计，先使用 TakeFirstLine 取出数据集里面的代码数据，接着调用 BuildRegexText 魔法函数让 GPT 生成正则表达式字符串，最后调用 RegexMatchText 函数使用 GPT 的正则表达式字符串处理数据集

这就是本文的实现的所有代码了，这个代码可以适配非常多的不同格式的数据，只要 GPT 魔法函数 BuildRegexText 能够正常输出正确的正则表达式的，那以上代码都能符合预期工作

接下来测试一下，看看以上代码能否符合预期工作

```
var data1 =
"""
在2023年9月1号开始上课
在2023年9月2号开始准备教材
在2023年9月3号完成作业
""";

var data2 =
"""
在9.1.2023开始上课
在9.2.2023开始准备教材
在9.3.2023完成作业
""";

Console.WriteLine($"开始执行解析 data1");
await RunAsync(data1);
Console.WriteLine($"开始执行解析 data2");
await RunAsync(data2);
```

可以看到，以上代码给出的是两个日期格式不相同的数据集，这时候就可以看到 LLM 的威力了。运行代码，可以看到控制台有以下输出

```
开始执行解析 data1
2023年9月1号
2023年9月2号
2023年9月3号

开始执行解析 data2
9.1.2023
9.2.2023
9.3.2023
```

可以看到正确输出了两个数据集的日期

也就是说尽管两个数据集采用不同的日期表达形式，但都在咱以上代码能工作的范围内，大家也可以试试更加奇怪的数据集，看是否能够符合预期工作

这就是 SemanticKernel 威力，使用 LLM 配合传统编程语言函数完成工作，发挥 LLM 和传统编程语言的优势

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/db13740804d16b3c56e8c24ab5a9ddf40962ecec/WalhocebarkairQalbehoreho) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/db13740804d16b3c56e8c24ab5a9ddf40962ecec/WalhocebarkairQalbehoreho) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin db13740804d16b3c56e8c24ab5a9ddf40962ecec
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin db13740804d16b3c56e8c24ab5a9ddf40962ecec
```

获取代码之后，进入 WalhocebarkairQalbehoreho 文件夹

---
title: dotnet 基于 DirectML 控制台运行 Phi-3 模型
description: 本文将和大家介绍如何在 C# dotnet 里面的控制台应用里面，使用 DirectML 将 Phi-3 模型在本地运行起来
tags: dotnet
category: 
---

<!-- CreateTime:2024/06/13 07:27:21 -->

<!-- 发布 -->
<!-- 博客 -->

在微软的 [Microsoft Build 2024](https://azure.microsoft.com/en-us/blog/new-models-added-to-the-phi-3-family-available-on-microsoft-azure/) 大会上介绍了 Phi-3 模型，这是一个 small language models (SLMs) 本地小语言模型。简单说就是一个可以在用户设备上运行的模型，据说能和 Gpt 3.5 进行 PK 的模型，不仅体积较小，且运行速度较快

在[上一篇](https://blog.lindexi.com/post/WPF-%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8-WinML-%E5%81%9A%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E6%89%8B%E5%86%99%E6%95%B0%E5%AD%97%E8%AF%86%E5%88%AB%E5%BA%94%E7%94%A8.html)博客和大家介绍了 WinML 和 DirectML 的基础信息。基于 DirectML 可以更加方便的在用户机器上部署 Phi-3 模型，简单到直接将模型文件拷贝过去就可以运行。通过 DirectML 屏蔽底层运行细节，可以在特别多的机器型号上运行，即使 GPU 不支持，还可以自动降级使用 CPU 运行

基于 DirectML 的优势就在于可以使用 DirectML 屏蔽大量底层细节，简化模型部署工作，且能够充分利用机器设备资源

更多关于 Phi-3 的介绍请参阅 <https://azure.microsoft.com/en-us/blog/introducing-phi-3-redefining-whats-possible-with-slms/>

在开始之前，需要大家从 <https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-onnx/tree/main?clone=true> 下载仓库，大概的下载命令如下

```
git lfs install

git clone https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-onnx
```

前置需要下载好了 git-lfs 工具，可到 <https://git-lfs.com> 官网进行下载。需要这个工具的原因是模型本身是通过 git lfs 使用 git 管理的。模型文件非常大，需要使用 git lfs 进行下载

下载的仓库大小大概有 20GB 左右，如果大家实在拉不下来，可以邮件给我，我将通过网盘分享给大家

下载下来的仓库有多个不同的版本，在本文例子里面将使用的是 DirectML 版本，即需要取出 directml-int4-awq-block-128 文件夹里面的所有文件，将其拷贝的最终应用的输出文件夹，或者自己找个文件夹放着。如我就将其拷贝到 `C:\lindexi\Phi3\directml-int4-awq-block-128\` 文件夹，拷贝之后的文件夹里面的文件内容如下

```
C:\lindexi\Phi3\
├── directml-int4-awq-block-128
│   ├── added_tokens.json
│   ├── genai_config.json
│   ├── model.onnx
│   ├── model.onnx.data
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   ├── tokenizer_config.json
```

完成基础下载模型文件之后，接下来咱来开始编写一个 dotnet 控制台应用。其实对于 dotnet 系应用来说，控制台能跑了，基本上意味着搭配上层 UI 框架也都能跑，比如上层 UI 框架使用 WPF 或 WinUI 或 MAUI 等框架都是可以的。本文使用控制台只是为了简单方便起见

新建 dotnet 控制台项目，编辑 csproj 文件用于安装 `Microsoft.ML.OnnxRuntimeGenAI.DirectML` 库，编辑之后的 csproj 代码如下

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
  </ItemGroup>
</Project>
```

完成基础准备之后，接下来可以进行编写核心逻辑

总的基于 DirectML 使用本地 Phi-3 模型的步骤如下

- 加载模型
- 构建输入信息
- 执行思考和输出

加载模型信息的代码很少，只需要创建 `Microsoft.ML.OnnxRuntimeGenAI.Model` 对象即可，如以下代码

```csharp
using Microsoft.ML.OnnxRuntimeGenAI;

using System.Text;

var folder = @"C:\lindexi\Phi3\directml-int4-awq-block-128\";

using var model = new Model(folder);
```

以上的 folder 文件夹里面存放的是我本地的 Phi-3 模型文件的路径，请大家修改为自己的实际使用路径

接下来再使用 `Microsoft.ML.OnnxRuntimeGenAI.Model` 对象创建出 `Microsoft.ML.OnnxRuntimeGenAI.Tokenizer` 对象。这里的 Tokenizer 是将文本转换为机器友好的 Token 符号的作用，由于直接输入人类的文本对于机器来说不够友好，才有了这一步

```csharp
using var tokenizer = new Tokenizer(model);
```

完成了模型的加载之后，接下来将通过控制台获取用户的输入内容，构建输入信息

```csharp
    Console.WriteLine("请输入聊天内容");

    var text = Console.ReadLine();

    var prompt = text;
```

以上的代码里面直接使用控制台输入的内容作为提示词信息，这样做比较简单，但实际的效果将会让 Phi-3 模型完全作为填充完成的存在。即 Phi-3 将尝试补全输入的文本后续的内容

如想要有一个更好的提示词效果，可以使用如下字符串方式进行填充

```csharp
            var prompt = $@"<|system|>{systemPrompt}<|end|><|user|>{userPrompt}<|end|><|assistant|>";
```

为了简单起见，本文只采用用户输入信息作为提示词。本文只是让大家能够将 Phi-3 模型跑起来，至于模型输出效果，那就看大家自己炼丹了

获取到提示词之后，需要使用上文创建的 tokenizer 将其转换为 token 列表，这里的 token 列表其实就是一个数字集合。简单理解就是一个给机器友好的字符串编码过程而已，专业术语是 词元化（Tokenization）过程

```csharp
var sequences = tokenizer.Encode(prompt);
```

将获取到的 token 列表进行构建输入参数

```csharp
    var generatorParams = new GeneratorParams(model);

    generatorParams.SetSearchOption("max_length", 1024);
    generatorParams.SetInputSequences(sequences);
    generatorParams.TryGraphCaptureWithMaxBatchSize(1);
```

将输入参数传递给到 `Microsoft.ML.OnnxRuntimeGenAI.Generator` 对象，代码如下

```csharp
    using var generator = new Generator(model, generatorParams);
```

接下来即可使用 `generator.ComputeLogits` 方法让模型进入思考状态，以及通过 GenerateNextToken 方法生成模型所输出的 token 内容

由于模型的输出也是一个 token 内容，不是人类优化的文本，此时就需要再使用 tokenizer 的 Decode 方法将 token 转换为文本

这里有一个坑点在于不是每一个 token 都能对应一个单词，有些是需要多个 token 才能对应一个单词。为了方便开发者，微软提供了 Microsoft.ML.OnnxRuntimeGenAI.TokenizerStream 类型，支持一个个 token 传入。自动处理多个 token 对应一个单词的情况。使用方法就是不断将模型生成的 token 传入给到 TokenizerStream 里，如果 TokenizerStream 判断输入的 token 足够生成单词了，就会返回单词字符串，否则将会返回空字符串。举个例子，如果有个单词需要三个 token 才能生成，那在传入给到 TokenizerStream 第一个和第二个 token 时，都会返回空字符串，传入第三个 token 时才会返回单词字符串

创建 TokenizerStream 的代码如下

```csharp
    using TokenizerStream tokenizerStream = tokenizer.CreateStream();
```

由于模型不是一次思考就能完成的，每次思考只是算出下一个 token 而已，需要编写一个循环等待模型完成

```csharp
    while (!generator.IsDone())
    {
        ... // 忽略其他代码
    }
```

进入循环，先调用 ComputeLogits 进行思考，再调用 GenerateNextToken 获取模型创建的下一个 token 内容

```csharp
    while (!generator.IsDone())
    {
        generator.ComputeLogits();
        generator.GenerateNextToken();

        ... // 忽略其他代码
    }
```

模型生成的下一个 token 都会自动追加到模型的 Sequence 里面。而 Sequence 可以认为就是拼接了输入的内容，也就是说模型将在输入的内容的基础上继续追加 token 内容

```csharp
    while (!generator.IsDone())
    {
        generator.ComputeLogits();
        generator.GenerateNextToken();

        // 这里的 tokenSequences 就是在输入的 sequences 后面添加 Token 内容
        var tokenSequences = generator.GetSequence(0);

        ... // 忽略其他代码
    }
```

此时拿到的 tokenSequences 就是在输入的 sequences 后面添加 Token 内容。可以使用 Tokenizer 的 Decode 方法将其转换为人类可读的文本

```csharp
        // 当前全部的文本
        var allText = tokenizer.Decode(tokenSequences);
```

这里转换到的是全部的文本内容，包括了输入的内容以及模型每次思考创建的内容

如果只是想要实现获取模型每一次思考时创建的内容，即实现一个词一个词输出，则需要使用 TokenizerStream 辅助，代码如下

```csharp
        // 这里的 tokenSequences 就是在输入的 sequences 后面添加 Token 内容
        var tokenSequences = generator.GetSequence(0);

        // 每次只会添加一个 Token 值
        // 需要调用 tokenizerStream 的解码将其转为人类可读的文本
        // 由于不是每一个 Token 都对应一个词，因此需要根据 tokenizerStream 压入进行转换，而不是直接调用 tokenizer.Decode 方法，或者调用 tokenizer.Decode 方法，每次都全部转换

        // 取最后一个进行解码为文本
        var decodeText = tokenizerStream.Decode(tokenSequences[^1]);
```

只需将 decodeText 在控制台输出，即可看到控制台不断一个个词进行输出

```csharp
        Console.Write(decodeText);
```

可以看到这个过程实现的代码很少，本文使用的 Program.cs 的全部代码如下

```csharp
// See https://aka.ms/new-console-template for more information

using Microsoft.ML.OnnxRuntimeGenAI;

using System.Text;

var folder = @"C:\lindexi\Phi3\directml-int4-awq-block-128\";
if (!Directory.Exists(folder))
{
    folder = Path.GetFullPath(".");
}

using var model = new Model(folder);
using var tokenizer = new Tokenizer(model);

for(var i = 0; i < int.MaxValue; i++)
{
    Console.WriteLine("请输入聊天内容");

    var text = Console.ReadLine();

    var prompt = text;

    var generatorParams = new GeneratorParams(model);

    var sequences = tokenizer.Encode(prompt);

    generatorParams.SetSearchOption("max_length", 1024);
    generatorParams.SetInputSequences(sequences);
    generatorParams.TryGraphCaptureWithMaxBatchSize(1);

    using var tokenizerStream = tokenizer.CreateStream();
    using var generator = new Generator(model, generatorParams);
    StringBuilder stringBuilder = new();

    while (!generator.IsDone())
    {
        generator.ComputeLogits();
        generator.GenerateNextToken();

        // 这里的 tokenSequences 就是在输入的 sequences 后面添加 Token 内容
        var tokenSequences = generator.GetSequence(0);

        // 每次只会添加一个 Token 值
        // 需要调用 tokenizerStream 的解码将其转为人类可读的文本
        // 由于不是每一个 Token 都对应一个词，因此需要根据 tokenizerStream 压入进行转换，而不是直接调用 tokenizer.Decode 方法，或者调用 tokenizer.Decode 方法，每次都全部转换

        // 当前全部的文本
        var allText = tokenizer.Decode(tokenSequences);

        // 取最后一个进行解码为文本
        var decodeText = tokenizerStream.Decode(tokenSequences[^1]);
        // 有些时候这个 decodeText 是一个空文本，有些时候是一个单词
        // 空文本的可能原因是需要多个 token 才能组成一个单词
        // 在 tokenizerStream 底层已经处理了这样的情况，会在需要多个 Token 才能组成一个单词的情况下，自动合并，在多个 Token 中间的 Token 都返回空字符串，最后一个 Token 才返回组成的单词
        if (!string.IsNullOrEmpty(decodeText))
        {
            stringBuilder.Append(decodeText);
        }
        Console.Write(decodeText);
    }

    Console.WriteLine("完成对话");
}

Console.WriteLine("Hello, World!");
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c7700766b617586eccb090ba859557ef08817484/Bp/FaldekallroCigerlurbe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c7700766b617586eccb090ba859557ef08817484/Bp/FaldekallroCigerlurbe) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c7700766b617586eccb090ba859557ef08817484
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c7700766b617586eccb090ba859557ef08817484
```

获取代码之后，进入 Bp/FaldekallroCigerlurbe 文件夹，即可获取到源代码

将下载的 Phi-3 模型的文件放入到一个文件夹，修改 folder 变量使用你自己本机的 Phi-3 模型文件夹路径，运行代码，在控制台输入你想和 Phi-3 模型交互的提示词，即可看到 Phi-3 模型的输出内容

这个过程可以配合打开任务管理器，看看自己设备的 CPU 和 GPU 的运行情况

如果想要发布给到其他伙伴运行，可以将模型文件放入到你的项目输出文件夹里面，这样即可让其他伙伴运行。如此也可以看到此方式的部署是非常简单的，不需要额外部署复杂的环境，只需要拷贝文件过去即可

本文实际使用的 Microsoft.ML.OnnxRuntimeGenAI.DirectML 还是预览版，也许后续正式版本将会更改一些内容

尽管本文演示的是控制台方式运行，但大家可以非常方便在此基础上构建一个 UI 界面，欢迎大家在此基础上制作自己的应用


<!-- 造了一个有趣的东西，基于 DirectML 控制台运行本地小语言模型 Phi-3 的应用。这是我构建好的包，这是一个分卷的 7z 文件，下载之后，使用 7z 打开即可。如果下载之后没有正确命名文件，则需要自己改一下

zip.zip.001 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwnxwyhhwljhjxhnillmyhnjphihhh
zip.zip.002 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwjpynhhwljhjxhnillnhxiihhihhh
zip.zip.003 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwkpmyhhwljhjxhnillnkqivhhihhh
zip.zip.004 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwkpmyhhwljhjxhnillnnmivhhihhh
zip.zip.005 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwimyzhhwljhjxhnillnqikkphihhh
zip.zip.006 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwvnjwhhwljhjxhnillnwjjpphihhh
zip.zip.007 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwkqkzhhwljhjxhnillnyylhhhihhh
zip.zip.008 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwkxuxhhwljhjxhnilloivlohhihhh
zip.zip.009 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwimyzhhwljhjxhnillololwphihhh
zip.zip.010 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwppklhhwljhjxhnillooknphhihhh
zip.zip.011 : https://ali-pro-pub.xbstatic.com/running-wechat-mp/uwiwjpynhhwljhjxhnilloqzpxphihhh

解压缩之后，双击 FaldekallroCigerlurbe.exe 即可运行 -->

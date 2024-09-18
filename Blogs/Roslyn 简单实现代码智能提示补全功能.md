---
title: Roslyn 简单实现代码智能提示补全功能
description: 相信有很多伙伴热衷于编写 IDE 应用，在 dotnet 系下，通过 Roslyn 友好的 API 和强大的能力，实现一个代码智能提示是非常简单的事情。本文将和大家简单介绍一下如何使用 Roslyn 实现简单的代码智能提示补全功能

<!--more-->

tags: Roslyn MSBuild 编译器
category: 
---

<!-- CreateTime:2024/08/18 07:25:31 -->
<!-- 置顶2 -->
<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器 -->

现在的 dotnet C# 核心构建工具链是非常完善的且开放的，基于 dotnet 完善的构建核心能力，咱可以非常方便的在此基础之上构建咱的编译器相关的应用

由于如何制作一个 IDE 应用是一个庞大的话题，本文仅仅只是和大家介绍如何使用 Roslyn 实现简单的代码智能提示补全功能。本文实现的示例代码是全控制台的演示，没有涉及任何界面逻辑，大家可以在本文末尾找到本文使用的代码的下载方法

本文属于 [Roslyn系列博客](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )，前置知识还请大家自行从 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 了解

先介绍一下本文预期实现的功能：当输入 `Cons` 代码时，预期输入 "Cons" 能够得到 Console 等单词的补全。当然了，在阅读完成本文之后，相信大家也可以非常方便的换成自己的其他代码片段获取其补全信息

通过 Roslyn 实现智能补全的核心是通过 Microsoft.CodeAnalysis.Completion.CompletionService 的 GetCompletionsAsync 方法获取补全列表

本文提供的方法比 [Roslyn如何实现简单的代码提示 - JackWang-CUMT - 博客园](https://www.cnblogs.com/isaboy/p/RoslynCodeCompletionSample.html ) 博客使用的方法更加具有代码通用性，可以实现近似于 VisualStudio 里的智能提升功能，甚至如果大家考虑自己编写一点排序算法，还可以做到近似于 ReSharper 的功能

以下是从从零开始编写代码，先创建一个控制台项目，我这里创建的是 .NET 9 的控制台项目。编辑 csproj 项目文件为以下代码用于安装必备的 NuGet 库

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Scripting" Version="4.9.2" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Features" Version="4.9.2" />
    <PackageReference Include="Microsoft.CodeAnalysis.Workspaces.MSBuild" Version="4.9.2" />
  </ItemGroup>
</Project>
```

为了方后续代码的编写方便，打开 Program.cs 文件，先添加一些命名空间引用

```csharp
using System.Diagnostics;

using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.Completion;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.Host.Mef;
using Microsoft.CodeAnalysis.Text;
```

完成引用之后，接下来就是开始创建虚拟的解决方案 Solution 和项目了。创建虚拟的解决方案不是要大家真的去新建一个 sln 文件，只需使用如下代码即可完成创建

```csharp
var adhocWorkspace = new AdhocWorkspace(MefHostServices.DefaultHost);
Solution solution = adhocWorkspace.CurrentSolution;
```

当然了，此时的 Solution 里面一个项目都没有，需要咱开始创建一个虚拟的项目才能加入到 Solution 里

创建项目时，重要的一点是加入引用程序集，引用程序集是非常重要的步骤。如果缺少这一步骤，将会导致找不到依赖，进而导致后续的智能提示等逻辑失败

本文这里只是添加必备的引用程序集，代码如下

```csharp
// 加上引用程序集，防止找不到引用
var referenceAssemblyPaths = new[]
{
    typeof(object).Assembly.Location,
    typeof(Console).Assembly.Location,
};
```

大家可以根据自己的实际需求选择所添加的引用程序集。关于引用程序集等属于 dotnet 基础知识，还请自行参阅 dotnet 基础知识

完成引用程序集收集之后，接下来就是通过 CSharpCompilationOptions 创建项目，代码如下

```csharp
var csharpCompilationOptions = new CSharpCompilationOptions
(
    OutputKind.DynamicallyLinkedLibrary, // 输出类型 dll 类型
    usings: new[] { "System" }, // 引用的命名空间
    allowUnsafe: true, // 允许不安全代码
    sourceReferenceResolver: new SourceFileResolver
    (
        searchPaths: new[] { Environment.CurrentDirectory },
        baseDirectory: Environment.CurrentDirectory
    )
);

var project = ProjectInfo.Create(ProjectId.CreateNewId(), VersionStamp.Create(),
    name: "Lindexi",
    assemblyName: "Lindexi",
    language: csharpCompilationOptions.Language,
    metadataReferences: referenceAssemblyPaths.Select(t => MetadataReference.CreateFromFile(t)));
```

再将项目添加到 Solution 里。由于 Roslyn 信奉的是不可变，调用了 AddProject 之后将会返回新的 Solution 对象，原有的 Solution 不被改变，因此需要重新赋值，代码如下

```csharp
solution = solution.AddProject(project);
```

新建的项目里面还没有代码，咱继续创建代码文档，代码如下

```csharp
var documentInfo = DocumentInfo.Create(DocumentId.CreateNewId(project.Id), name: "LindexiCode", sourceCodeKind: SourceCodeKind.Script);
```

将文档直接添加到解决方案里面，用于获取从 DocumentInfo 获取到 Microsoft.CodeAnalysis.Document 对象

```csharp
solution = solution.AddDocument(documentInfo);
```

依然是不可变思想的写法，调用 AddDocument 之后会返回新的 Solution 对象，需要重新赋值

加入到 Solution 之后，即可通过 GetDocument 获取到 Document 对象用于后续分析

```csharp
Document document = solution.GetDocument(documentInfo.Id)!;
```

接下来咱尝试模拟输入的代码，预期输入 "Cons" 能够得到 Console 补全

```csharp
var text = "Cons";
```

将此模拟的代码放入到文档里面，代码如下

```csharp
Document textDocument = document.WithText(SourceText.From(text));
```

现在基于此文档，在分析器看来的代码大概如下

```csharp
using System;

Cons
```

以上代码的 `using System;` 是在创建项目的 CSharpCompilationOptions 添加的默认引用

通过文档关联关系获取到代码补全服务，获取方法如下

```csharp
CompletionService completionService = CompletionService.GetService(document)!;
Debug.Assert(completionService != null);
```

现在此文档已经可以被 Roslyn 进行分析了，尝试获取补全列表，代码如下

```csharp
CompletionList completionList = await completionService.GetCompletionsAsync(textDocument, caretPosition: text.Length);
```

以上就是本文的核心代码逻辑。接下来就是需要对补全列表进行排序，补全列表的内容如果太多且没有排序的话，那开发者看着补全列表也不开森，以下是我使用简单的排序方法进行排序，然后在控制台输出的代码

```csharp
foreach (var completionItem in completionList.ItemsList
             .OrderBy(item => item.DisplayText.StartsWith(text) ? 0 : 1)
             .ThenByDescending(item => item.Rules.MatchPriority)
             .ThenBy(item => item.SortText))
{

    Console.WriteLine($"""
                       DisplayText:{completionItem.DisplayText}
                       SortText:{completionItem.SortText}
                       FilterText:{completionItem.FilterText}
                       MatchPriority:{completionItem.Rules.MatchPriority}
                       
                       """);
}
```

我先排序的是包含输入的代码字符的，于是 `Consistency` 和 `Console` 就可以排在前面，接着再使用通用的 MatchPriority 和 SortText 加入到排序里面

大概的输出内容如下

```csharp
DisplayText:Consistency
SortText:~Consistency  System.Runtime.ConstrainedExecution
FilterText:Consistency
MatchPriority:0

DisplayText:Console
SortText:~Console  System
FilterText:Console
MatchPriority:0

DisplayText:Console
SortText:~Console Internal
FilterText:Console
MatchPriority:0

...
```

通过 CompletionItem 可以获取到补全的内容的很多信息，包括这个提示是关键词还是类型还是代码片等等

我感觉 ReSharper 的智能提示比 VisualStudio 更好用的部分，不在于智能提示的内容，而在于智能提示的内容的排序上，经常我可以发现 ReSharper 将我需要的东西排在前面甚至第一个，然而 VisualStudio 经常将我不需要的内容放在前面。如果大家想要实现一个 IDE 应用，我感觉需要智能提示和补全倒是问题不大，比较期望花精力的是在于智能提示的内容的排序

以上就是本文演示的使用 CompletionService 提供的智能补全功能，通过此功能可以辅助大家更加方便实现智能代码补全和智能提示功能

至于如何使用此能力去制作一个属于自己的 IDE 那就需要大家自行编写了

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c5eebfd01f7b4c567c2719161b7513e2e7b6df06/Workbench/WhinerfejuwhawHallkeferbai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c5eebfd01f7b4c567c2719161b7513e2e7b6df06/Workbench/WhinerfejuwhawHallkeferbai) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c5eebfd01f7b4c567c2719161b7513e2e7b6df06
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c5eebfd01f7b4c567c2719161b7513e2e7b6df06
```

获取代码之后，进入 Workbench/WhinerfejuwhawHallkeferbai 文件夹，即可获取到源代码

更多 Roslyn 相关博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

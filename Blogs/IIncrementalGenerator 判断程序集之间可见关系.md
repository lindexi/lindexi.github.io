---
title: IIncrementalGenerator 增量 Source Generator 生成代码入门 判断程序集之间的 InternalsVisibleTo 关系
description: 本文告诉大家如何在使用 IIncrementalGenerator 进行增量的 Source Generator 生成代码时，如何判断两个程序集之间是否存在 InternalsVisibleTo 关系

<!--more-->

tags: Roslyn MSBuild 编译器 SourceGenerator 生成代码
category: 
---

<!-- CreateTime:2023/6/19 8:39:59 -->
<!-- 标题： IIncrementalGenerator 判断程序集之间可见关系 -->
<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

当获取到两个程序集时，如果要开始准备生成相关代码，可能会因为不知道两个程序集之间是否存在 InternalsVisibleTo 关系，也就是是否应该导出其 internal 的类型而困扰。在能够获取到 IAssemblySymbol 类型的对象，即可通过 GivesAccessTo 方法判断两个程序集的 InternalsVisibleTo 关系

这个 GivesAccessTo 方法可以获取到当前的程序集对给定的程序集参数是否为 internal 可见

以下是详细的例子代码

本文的例子的任务是编写一个 Roslyn 分析器，在分析器里面使用 IIncrementalGenerator 增量 Source Generator 生成代码，获取到对当前正在分析的项目设置 InternalsVisibleTo 的引用程序集，将程序集名作为生成代码的部分，让正在被分析的项目可以编写代码输出有哪些程序集是 internal 可见的

先新建以下 .NET 7 控制台项目，分别是名为 Analyzers 和 App 和 Lib1 和 Lib2 项目

在 Lib1 和 Lib2 里面存放一些 internal 的类型，这两个项目将被当成类库项目被 App 项目所引用。在 Lib2 里面添加一个 AssemblyInfo.cs 文件，在 AssemblyInfo.cs 文件里面记录 InternalsVisibleTo 给到 App 程序集，如以下代码。于是 Lib1 没有对 App 项目 internal 可见，而 Lib2 对 App 项目 internal 可见

```csharp
using System.Runtime.CompilerServices;

[assembly: InternalsVisibleTo("App")]
```

设置 App 项目引用 Lib1 和 Lib2 项目，且引用 Analyzers 项目作为分析器项目。完成设置的 App 项目的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Lib1\Lib1.csproj" />
    <ProjectReference Include="..\Lib2\Lib2.csproj" />
    <ProjectReference Include="..\Analyzers\Analyzers.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
  </ItemGroup>

</Project>
```

大概的项目组织如下图

![](http://cdn.lindexi.site/lindexi%2F20236171619301553.jpg)

修改 Analyzers 项目，让这个项目成为 Roslyn 分析器项目，修改之后的 csproj 项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>

    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
    <LangVersion>latest</LangVersion>

    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.2.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

这里的 TargetFramework 设置为 netstandard2.0 是为了同时让 VisualStudio 和 dotnet 开森

详细关于以上 csproj 项目文件代码里的 EnforceExtendedAnalyzerRules 的属性，请参阅 [Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用](http://blog.lindexi.com/post/Roslyn-%E5%88%86%E6%9E%90%E5%99%A8-EnforceExtendedAnalyzerRules-%E5%B1%9E%E6%80%A7%E7%9A%84%E4%BD%9C%E7%94%A8.html)

以上的 LangVersion 属性设置为 latest 表示使用最新的语言版本，详细请参阅 [VisualStudio 使用三个方法启动最新 C# 功能](https://blog.lindexi.com/post/VisualStudio-%E4%BD%BF%E7%94%A8%E4%B8%89%E4%B8%AA%E6%96%B9%E6%B3%95%E5%90%AF%E5%8A%A8%E6%9C%80%E6%96%B0-C-%E5%8A%9F%E8%83%BD.html )

通过以上配置即可完成项目的初始化逻辑。回到咱这个例子的任务上，就是在 Analyzers 分析器项目编写代码，分析 App 项目所引用的程序集里面的存在哪些程序集对 App 程序集设置了 internal 可见

完成准备工作之后，接下来开始本文的核心逻辑编写。先在 Analyzers 分析器项目上新建一个继承 IIncrementalGenerator 接口的 FooTelescopeIncrementalGenerator 类型，接下来的核心逻辑将在 FooTelescopeIncrementalGenerator 的 Initialize 开始编写

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Microsoft.CodeAnalysis;

namespace Analyzers;

[Generator(LanguageNames.CSharp)]
public class FooTelescopeIncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
       ... // 忽略代码
    }
}
```

根据上文的描述，咱需要先从 context 里面的 CompilationProvider 获取到引用的程序集，代码如下

```csharp
[Generator(LanguageNames.CSharp)]
public class FooTelescopeIncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var internalsVisibleFromAssemblyNameListIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            ... // 忽略代码
        });
    }
}
```

通过 compilation 的 SourceModule 属性的 ReferencedAssemblySymbols 即可获取到所有的引用程序集，如以下代码

```csharp
[Generator(LanguageNames.CSharp)]
public class FooTelescopeIncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var internalsVisibleFromAssemblyNameListIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            ... // 忽略代码
        });
    }
}
```

从 compilation 里面拿到的 Assembly 属性就是当前正在分析的程序集，在本文这里就是 App 程序集。而 `referencedAssemblySymbols` 里面都是当前的 App 程序集所引用的程序集。判断引用的程序集是否对当前正在分析的程序集设置了 internal 可见，即可通过 GivesAccessTo 方法进行判断，代码如下

```csharp
[Generator(LanguageNames.CSharp)]
public class FooTelescopeIncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var internalsVisibleFromAssemblyNameListIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            var internalsVisibleFromAssemblyNameList = new List<string>();

            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            foreach (var referencedAssemblySymbol in referencedAssemblySymbols)
            {
                var name = referencedAssemblySymbol.Name;

                if (referencedAssemblySymbol.GivesAccessTo(compilation.Assembly))
                {
                    internalsVisibleFromAssemblyNameList.Add(name);
                }
            }

            return internalsVisibleFromAssemblyNameList;
        });

        ... // 忽略代码
    }
}
```

接下来将收集到的给当前正在分析的程序集设置了 internal 可见的程序集列表输出到生成代码里面，如以下代码

{% raw %}

```csharp
using System.Collections.Generic;
using System.Linq;
using Microsoft.CodeAnalysis;

namespace Analyzers;

[Generator(LanguageNames.CSharp)]
public class FooTelescopeIncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var internalsVisibleFromAssemblyNameListIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            var internalsVisibleFromAssemblyNameList = new List<string>();

            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            foreach (IAssemblySymbol? referencedAssemblySymbol in referencedAssemblySymbols)
            {
                var name = referencedAssemblySymbol.Name;

                if (referencedAssemblySymbol.GivesAccessTo(compilation.Assembly))
                {
                    internalsVisibleFromAssemblyNameList.Add(name);
                }
            }

            return internalsVisibleFromAssemblyNameList;
        });

        context.RegisterSourceOutput(internalsVisibleFromAssemblyNameListIncrementalValueProvider, (productionContext, list) =>
        {
            var code = $@"
    public static class InternalsVisibleToHelper
    {{
        public static IEnumerable<string> GetAllInternalsVisibleFromAssemblyName()
        {{
            {(string.Join("\r\n", list.Select(t => $@"yield return ""{t}"";")))}
        }}
    }}";
            productionContext.AddSource("InternalsVisibleToHelper", code);
        });
    }
}
```

{% endraw %}

回到 App 项目里面，编辑 Program.cs 文件，输出以上生成的 InternalsVisibleToHelper 类型里面的 GetAllInternalsVisibleFromAssemblyName 方法返回内容到控制台，如以下代码

```csharp
foreach (var name in InternalsVisibleToHelper.GetAllInternalsVisibleFromAssemblyName())
{
    Console.WriteLine(name);
}
```

运行 App 项目，可以看到控制台很符合预期的只输出了 Lib2 程序集

通过以上的代码，即可在 Roslyn 分析器里面，了解程序集之间的 internal 关系，从而可以生成出更加符合预期的代码

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e0748230af39e712b77e72f2dbb6bef4453b0c84/QahayhebeeJaydearlenayjeadel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e0748230af39e712b77e72f2dbb6bef4453b0c84/QahayhebeeJaydearlenayjeadel) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e0748230af39e712b77e72f2dbb6bef4453b0c84
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e0748230af39e712b77e72f2dbb6bef4453b0c84
```

获取代码之后，进入 QahayhebeeJaydearlenayjeadel 文件夹

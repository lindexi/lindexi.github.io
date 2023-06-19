# IIncrementalGenerator 增量 Source Generator 生成代码入门 获取引用程序集的所有类型

本文告诉大家如何在使用 IIncrementalGenerator 进行增量的 Source Generator 生成代码时，如何获取到当前正在分析的程序集所引用的所有的程序集，以及引用的程序集里面的所有类型

<!--more-->
<!-- CreateTime:2023/6/19 8:39:59 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

这项技术可以用在生成导出类型相关的需求上，比如我想导出我当前程序集里面所有引用的程序集的继承于 `IFoo` 接口的所有类型，即可采用本文介绍的方法

核心逻辑是在 Compilation 里面拿到 SourceModule 属性，通过 IModuleSymbol 类型的 SourceModule 属性进一步拿到 `ImmutableArray<IAssemblySymbol>` 类型的 ReferencedAssemblySymbols 属性

这里的 ReferencedAssemblySymbols 属性就是当前的程序集所引用的程序集了

在这些程序集上枚举所有程序集内的语义类型即可获取到所有的类型

以下是详细的例子

为了方便描述本文的技术实现，需要创建三个项目，分别是 App 和 Lib 和 Analyzers 三个项目。在本文末尾将可以找到所有代码的下载方法

这里 App 项目是用来被分析器项目 Analyzers 项目进行分析的。而 Lib 项目则是一个基础库，被 App 项目所引用

在这个例子里面，咱的任务就是在 Analyzers 分析器项目里面编写代码，分析去 App 里面所引用的 Lib 项目里面包含的所有类型

具体的初始化方法就是新建三个 .NET 7 控制台项目，分别命名为 App 和 Lib 和 Analyzers 项目。接着编辑 Lib 项目修改为类库项目，修改的方法就是编辑 csproj 项目文件，替换为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

接着修改 App 项目的 csproj 项目文件，让 App 项目引用 Lib 项目以及 Analyzers 分析器项目。只有让 App 项目引用 Analyzers 分析器项目，才可以让 Analyzers 分析器项目对 App 项目进行分析，编辑之后的 csproj 项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Lib\Lib.csproj" />
    <ProjectReference Include="..\Analyzers\Analyzers.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
  </ItemGroup>

</Project>
```

如以上代码，需要让 App 项目引用 Analyzers 分析器项目，设置引用 OutputItemType 为 Analyzer 才可以让 Analyzers 项目被当成 App 项目的分析器

由于 App 项目不需要用到任何在 Analyzers 分析器项目定义的类型，于是也设置了 ReferenceOutputAssembly 为 false 值。通过 `OutputItemType="Analyzer" ReferenceOutputAssembly="false"` 两个属性即可让 Analyzers 项目只作为 App 项目的分析器存在，不影响 App 项目的其他逻辑，也不会让 App 项目真正引用到 Analyzers 项目里面的任何公开类型

同时设置了 App 项目引用 Analyzers 分析器项目，即可在构建的时候，先构建 Analyzers 分析器项目，再构建 App 项目，确定了项目的构建顺序。于是在 Analyzers 分析器项目里面编写的 IIncrementalGenerator 增量 Source Generator 生成代码逻辑将可以被正常执行

最后来到最重要的 Analyzers 分析器项目。为了能够让 VisualStudio 开森以及让 dotnet 开心，推荐使用的是 `netstandard2.0` 框架。然后引用上必要的 NuGet 包，修改之后的 csproj 项目文件代码如下

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

详细关于以上 csproj 项目文件代码里的 EnforceExtendedAnalyzerRules 的属性，请参阅 [Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用](http://blog.lindexi.com/post/Roslyn-%E5%88%86%E6%9E%90%E5%99%A8-EnforceExtendedAnalyzerRules-%E5%B1%9E%E6%80%A7%E7%9A%84%E4%BD%9C%E7%94%A8.html)

以上的 LangVersion 属性设置为 latest 表示使用最新的语言版本，详细请参阅 [VisualStudio 使用三个方法启动最新 C# 功能](https://blog.lindexi.com/post/VisualStudio-%E4%BD%BF%E7%94%A8%E4%B8%89%E4%B8%AA%E6%96%B9%E6%B3%95%E5%90%AF%E5%8A%A8%E6%9C%80%E6%96%B0-C-%E5%8A%9F%E8%83%BD.html )

通过以上配置即可完成项目的初始化逻辑。回到咱这个例子的任务上，就是在 Analyzers 分析器项目编写代码，分析 App 项目所引用的 Lib 项目里面的存在哪些类型

为了能够让 Analyzers 分析器项目有活干，咱就来给 Lib 项目多添加一些随意的类型

```csharp
namespace Lib;

public class Base
{
}

public class Foo1 : Base
{
}

public class Foo2 : IFoo
{
}

public class Foo3 : Base, IFoo
{
}

public interface IFoo
{
}
```

![](http://image.acmx.xyz/lindexi%2F20236171233417273.jpg)

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
        var typeNameIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
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
        var typeNameIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            ... // 忽略代码
        });
    }
}
```

遍历所有引用的程序集。这里为了博客方便，就只获取 Lib 程序集里面的类型

```csharp
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var typeNameIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            // 为了方便代码理解，这里只取名为 Lib 程序集的内容…
            foreach (var referencedAssemblySymbol in referencedAssemblySymbols)
            {
                var name = referencedAssemblySymbol.Name;

                if (name.Contains("Lib"))
                {
                    ... // 在这里编写获取程序集所有类型的代码
                }
                else
                {
                    // 其他的引用程序集，在这里就忽略
                }
            }
            ... // 忽略代码
        });
    }
```

通过 IAssemblySymbol 的 GlobalNamespace 属性即可获取到顶层的 INamespaceSymbol 符号，通过语义知识可以了解到，类型都是存放在命名空间里面的，只需要对命名空间进行递归即可获取到所有的类型

如以下代码即可递归获取某个 INamespaceSymbol 下的所有类型

```csharp
    private static IEnumerable<INamedTypeSymbol> GetAllTypeSymbol(INamespaceSymbol namespaceSymbol)
    {
        var typeMemberList = namespaceSymbol.GetTypeMembers();

        foreach (var typeSymbol in typeMemberList)
        {
            yield return typeSymbol;
        }

        foreach (var namespaceMember in namespaceSymbol.GetNamespaceMembers())
        {
            foreach (var typeSymbol in GetAllTypeSymbol(namespaceMember))
            {
                yield return typeSymbol;
            }
        }
    }
```

给 GetAllTypeSymbol 传入程序集 GlobalNamespace 即可获取到此程序集里面的所有类型，代码如下

```csharp
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var typeNameIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            // 为了方便代码理解，这里只取名为 Lib 程序集的内容…
            foreach (var referencedAssemblySymbol in referencedAssemblySymbols)
            {
                var name = referencedAssemblySymbol.Name;

                if (name.Contains("Lib"))
                {
                    // 获取所有的类型
                    // 这里 ToList 只是为了方便调试
                    var allTypeSymbol = GetAllTypeSymbol(referencedAssemblySymbol.GlobalNamespace).ToList();
                }
                else
                {
                    // 其他的引用程序集，在这里就忽略
                }
            }
            ... // 忽略代码
        });
    }
```

以上拿到的 `allTypeSymbol` 就是引用的 Lib 程序集里面的所有类型。为了测试咱的分析器代码是否正确，可以尝试将收集到的 Lib 程序集里面的所有类型的记录输出作为一个源代码生成

{% raw %}

```csharp
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        Debugger.Launch();

        var typeNameIncrementalValueProvider = context.CompilationProvider.Select((compilation, token) =>
        {
            var typeNameList = new List<string>();

            // 获取到所有引用程序集
            var referencedAssemblySymbols = compilation.SourceModule.ReferencedAssemblySymbols;

            // 为了方便代码理解，这里只取名为 Lib 程序集的内容…
            foreach (IAssemblySymbol? referencedAssemblySymbol in referencedAssemblySymbols)
            {
                var name = referencedAssemblySymbol.Name;

                if (name.Contains("Lib"))
                {
                    // 获取所有的类型
                    // 这里 ToList 只是为了方便调试
                    var allTypeSymbol = GetAllTypeSymbol(referencedAssemblySymbol.GlobalNamespace).ToList();

                    foreach (var typeSymbol in allTypeSymbol)
                    {
                        typeNameList.Add(typeSymbol.ToDisplayString());
                    }
                }
                else
                {
                    // 其他的引用程序集，在这里就忽略
                }
            }

            return typeNameList;
        });

        context.RegisterSourceOutput(typeNameIncrementalValueProvider, (productionContext, list) =>
        {
            var code = $@"
    public static class FooHelper
    {{
        public static IEnumerable<string> GetAllTypeName()
        {{
            {(string.Join("\r\n", list.Select(t => $@"yield return ""{t}"";")))}
        }}
    }}";
            productionContext.AddSource("FooHelper", code);
        });
    }
```

{% endraw %}

如以上代码就在代码生成器里面生成了名为 FooHelper 的类型，这个类型将会返回 Lib 程序集里面的所有的类型

接下来编辑 App 项目的 Program.cs 文件，替换为如下代码

```csharp
foreach (var name in FooHelper.GetAllTypeName())
{
    Console.WriteLine(name);
}
```

假设分析器项目代码编写正确，那就可以成功输出 Lib 程序集里面的所有类型到控制台

试试运行一下项目，看看写的对不对吧

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d0b5dc0af9c9f4ff3c18a2212200b492e3edbc08/LurwajedalwemcejemQallneleecairwhair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d0b5dc0af9c9f4ff3c18a2212200b492e3edbc08/LurwajedalwemcejemQallneleecairwhair) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d0b5dc0af9c9f4ff3c18a2212200b492e3edbc08
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin d0b5dc0af9c9f4ff3c18a2212200b492e3edbc08
```

获取代码之后，进入 LurwajedalwemcejemQallneleecairwhair 文件夹
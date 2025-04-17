
# IIncrementalGenerator 增量 Source Generator 生成代码入门 从语法到语义 获取类型完全限定名

本文告诉大家如何在使用 IIncrementalGenerator 进行增量的 Source Generator 生成代码时，如何从语法分析过程，将获取的语法 Token 转换到语义分析上，比如获取类型完全限定名。一个使用的例子是在拿到一个 Token 表示某个类型时，本文将演示通过语义分析获取到拿到的 Token 的 Type 类型的 FullName 带命名空间的完全限定名

<!--more-->


<!-- CreateTime:2023/6/2 16:15:05 -->

<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,生成代码 -->

本文属于 IIncrementalGenerator 增量 Source Generator 生成代码入门系列博客，前置的博客是 [尝试 IIncrementalGenerator 进行增量 Source Generator 生成代码](https://blog.lindexi.com/post/%E5%B0%9D%E8%AF%95-IIncrementalGenerator-%E8%BF%9B%E8%A1%8C%E5%A2%9E%E9%87%8F-Source-Generator-%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81.html )

更多编译器、代码分析、代码生成相关博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

在开始之前，期望大家已了解语法分析和语义分析的差别。可通过阅读 [Roslyn 入门：使用 Roslyn 静态分析现有项目中的代码（语法分析） - walterlv](https://blog.walterlv.com/post/analysis-code-of-existed-projects-using-roslyn.html ) 和 [使用 Roslyn 对 C# 代码进行语义分析 - walterlv](https://blog.walterlv.com/post/roslyn-semantic-analysis-starter ) 博客对此进行了解

## 初始化项目

在开始之前，先创建好测试使用的项目，创建两个项目，分别是分析器项目，和使用分析器的项目。项目分别是 WarnaijakeCiwhelwajifaje.Analyzers 和 WarnaijakeCiwhelwajifaje 两个项目

在 WarnaijakeCiwhelwajifaje.Analyzers 的 csproj 项目文件里面的代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>

    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>

    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.6.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

在 WarnaijakeCiwhelwajifaje 里引用分析器，项目文件里面的代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\WarnaijakeCiwhelwajifaje.Analyzers\WarnaijakeCiwhelwajifaje.Analyzers.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false"/>
  </ItemGroup>

</Project>
```

在 WarnaijakeCiwhelwajifaje 里添加 Program.cs 文件，在此文件放入 Main 方法，代码如下

```csharp
namespace WarnaijakeCiwhelwajifaje;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
    }
}
```

本文将演示通过语法分析拿到了 Program 定义，接着通过语义分析，拿到 Program 的完全限定名，也就是 `global::WarnaijakeCiwhelwajifaje.Program` 内容

## 创建分析器

接下来将在新建的分析器代码里面，先通过语法分析快速获取到 Program 的代码定义，接着在 SemanticModel 里面获取到 Program 类型的完全限定名

先新建继承 IIncrementalGenerator 的分析器类型，代码大概如下

```csharp
    [Generator(LanguageNames.CSharp)]
    public class FooIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            ... // 忽略代码
        }
    }
```
 
## 搭建解析框架

为了能够让代码调试跑起来，咱开始在 FooIncrementalGenerator 的 Initialize 方法里面编写一些调试辅助代码和搭建基础的框架代码

先加上 Debugger.Launch() 用于启动调试，加上这个代码即可在 WarnaijakeCiwhelwajifaje 项目构建时触发调试异常，从而可以被 VisualStudio 附加

```csharp
    [Generator(LanguageNames.CSharp)]
    public class FooIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            Debugger.Launch();
            ... // 忽略代码
        }
    }
```

接着通过 IncrementalGeneratorInitializationContext 的 SyntaxProvider 属性，调用 CreateSyntaxProvider 方法创建处理逻辑

此 CreateSyntaxProvider 方法接受两个委托函数，以下将告诉大家这两个委托函数参数的作用

通过 CreateSyntaxProvider 即可创建对所有参与构建的 cs 文件里面的代码逻辑的处理，在这里的思想是每次变更都是一个个进来的，变更的文件进来之后，将会先进入 CreateSyntaxProvider 方法传入的第一个委托参数，在这个委托参数里面将用来快速的语法判断，判断当前变更的文件是否在此业务逻辑上是感兴趣的。通过此快速判断逻辑即可过滤掉不需要处理的信息，从而减少后续需要处理的工作量，提升性能。在搭建的框架代码里面，只需要每次都返回 true 表示全部都需要处理即可

第二个委托参数就是转换处理逻辑，传入委托的是 GeneratorSyntaxContext 参数，要求返回值是处理完成返回的任意类型。在 GeneratorSyntaxContext 类型参数里面将包括语法的 Node 属性，和包括语义的 SemanticModel 属性，在框架代码里面只需要每次都返回 GeneratorSyntaxContext 参数即可

代码如下

```csharp
            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((syntaxNode, _) =>
                {
                    return true;
                },
                (GeneratorSyntaxContext generatorSyntaxContext, CancellationToken _) =>
                {
                    return generatorSyntaxContext;
                });
```

如果只有以上的代码，大家将会发现在委托里面添加断点将不会进入，这是因为 `incrementalValuesProvider` 没有被任何地方使用。在 Roslyn 里面的设计就是缓加载方式，和 Linq 一样，只有在需要用到的时候才执行

为了让以上的委托能够被执行，添加 RegisterSourceOutput 用来让底层执行委托内容，代码如下

```csharp
            context.RegisterSourceOutput(incrementalValuesProvider, (sourceProductionContext, generatorSyntaxContext) =>
            {
                // 加上这里只是为了让 incrementalValuesProvider 能够执行
            });
```

添加完成框架之后的代码如下

```csharp
using System.Diagnostics;
using System.Threading;

using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;

namespace WarnaijakeCiwhelwajifaje.Analyzers
{
    [Generator(LanguageNames.CSharp)]
    public class FooIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            Debugger.Launch();

            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((syntaxNode, _) =>
                {
                    return true;
                },
                (GeneratorSyntaxContext generatorSyntaxContext, CancellationToken _) =>
                {
                    return generatorSyntaxContext;
                });

            context.RegisterSourceOutput(incrementalValuesProvider, (sourceProductionContext, generatorSyntaxContext) =>
            {
                // 加上这里只是为了让 incrementalValuesProvider 能够执行
            });
        }
    }
}
```

接下来就可以开始修改框架的代码，逐个换成演示的代码

## 语法过滤

回到咱演示的主题，获取到 `Program` 代码对应的类型的完全限定名。从这个需求可以知道，咱感兴趣的语法一定是一个 class 类型定义，如此可以在 CreateSyntaxProvider 的第一个委托里面进行快速的语法过滤，过滤只有 ClassDeclaration 才允许进入后续逻辑，代码如下

```csharp
            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((syntaxNode, _) =>
                {
                    // 在此进行快速的语法判断逻辑，可以判断当前的内容是否感兴趣，如此过滤掉一些内容，从而减少后续处理，提升性能

                    // 这里样式的是获取到 Program 类的完全限定名，也就是只需要用到 Class 类型
                    
                    return syntaxNode.IsKind(SyntaxKind.ClassDeclaration);
                },
                (GeneratorSyntaxContext generatorSyntaxContext, CancellationToken _) =>
                {
                    return generatorSyntaxContext;
                });
```

## 从语法分析到语义分析

在 CreateSyntaxProvider 的第二个委托参数里面将用来编写处理逻辑，输入的参数 GeneratorSyntaxContext 类型里面包含了两个属性，分别是包括语法的 Node 属性，和包括语义的 SemanticModel 属性

先通过语法获取到 Program 类型定义，代码如下

```csharp
                    // 从这里可以获取到语法内容
                    if (generatorSyntaxContext.Node is ClassDeclarationSyntax classDeclarationSyntax)
                    {
                    }
                    else
                    {
                        // 理论上不会进入此分支，因为在之前判断了类型
                    }
```

这里的 Node 属性一定是 ClassDeclarationSyntax 类型，这是因为在前面语法部分限制了 `IsKind(SyntaxKind.ClassDeclaration)` 决定这里一定是类型定义

使用 SemanticModel 属性从语法 ClassDeclarationSyntax 获取到语义，代码如下

```csharp
var symbolInfo = generatorSyntaxContext.SemanticModel.GetDeclaredSymbol(classDeclarationSyntax)!;
```

如此即可完成从语法分析到语义分析。根据 [使用 Roslyn 对 C# 代码进行语义分析 - walterlv](https://blog.walterlv.com/post/roslyn-semantic-analysis-starter ) 博客的示例，可以了解到拿到 `symbolInfo` 对象之后，即可获取到当前语法 Program 对应的类型，约等于拿到反射的 Type 类型，即可方便获取到对应的命名空间，继承的类型，包含的成员等等

## 获取类型名

获取类型名的方法可以是让 `symbolInfo` 进行格式化输出，格式化输出可以定制输出格式，如以下代码

```csharp
                        // 带上 global 格式的输出 FullName 内容
                        var symbolDisplayFormat = new SymbolDisplayFormat
                        (
                            // 带上命名空间和类型名
                            SymbolDisplayGlobalNamespaceStyle.Included,
                            // 命名空间之前加上 global 防止冲突
                            SymbolDisplayTypeQualificationStyle
                                .NameAndContainingTypesAndNamespaces
                        );
```

调用 ToDisplayString 即可进行格式化输出定义

```csharp
var displayString = symbolInfo.ToDisplayString(symbolDisplayFormat);
```

通过以上代码就能拿到 `global::WarnaijakeCiwhelwajifaje.Program` 字符串

以下是分析器的代码

```csharp
    [Generator(LanguageNames.CSharp)]
    public class FooIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            Debugger.Launch();

            var incrementalValuesProvider = context.SyntaxProvider.CreateSyntaxProvider((syntaxNode, _) =>
                {
                    // 在此进行快速的语法判断逻辑，可以判断当前的内容是否感兴趣，如此过滤掉一些内容，从而减少后续处理，提升性能

                    // 这里样式的是获取到 Program 类的完全限定名，也就是只需要用到 Class 类型
                    
                    return syntaxNode.IsKind(SyntaxKind.ClassDeclaration);
                },
                (generatorSyntaxContext, _) =>
                {
                    // 从这里可以获取到语法内容
                    if (generatorSyntaxContext.Node is ClassDeclarationSyntax classDeclarationSyntax)
                    {
                        var symbolInfo = generatorSyntaxContext.SemanticModel.GetDeclaredSymbol(classDeclarationSyntax)!;
                        // 带上 global 格式的输出 FullName 内容
                        var symbolDisplayFormat = new SymbolDisplayFormat
                        (
                            // 带上命名空间和类型名
                            SymbolDisplayGlobalNamespaceStyle.Included,
                            // 命名空间之前加上 global 防止冲突
                            SymbolDisplayTypeQualificationStyle
                                .NameAndContainingTypesAndNamespaces
                        );
                        var displayString = symbolInfo.ToDisplayString(symbolDisplayFormat);
                    }
                    else
                    {
                        // 理论上不会进入此分支，因为在之前判断了类型
                    }

                    return generatorSyntaxContext;
                });

            context.RegisterSourceOutput(incrementalValuesProvider, (sourceProductionContext, generatorSyntaxContext) =>
            {
                // 加上这里只是为了让 incrementalValuesProvider 能够执行
            });
        }
    }
```

## 源代码

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d1197778d4a96524de210e44a662331e7340a720/WarnaijakeCiwhelwajifaje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d1197778d4a96524de210e44a662331e7340a720/WarnaijakeCiwhelwajifaje) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d1197778d4a96524de210e44a662331e7340a720
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin d1197778d4a96524de210e44a662331e7340a720
```

获取代码之后，进入 WarnaijakeCiwhelwajifaje 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
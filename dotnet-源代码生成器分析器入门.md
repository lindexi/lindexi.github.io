
# dotnet 源代码生成器分析器入门

本文将带领大家入门 dotnet 的 SourceGenerator 源代码生成器技术，期待大家阅读完本文能够看懂理解和编写源代码生成器和分析器

<!--more-->


<!-- CreateTime:2025/02/11 07:29:34 -->

<!-- 草稿 -->

恭喜你看到了本文，进入到 C# dotnet 的深水区。如果你还是在浅水玩耍的小鲜肉，推荐你点击右上方的关闭按钮，避免受到过于深入的知识的污染

在开始之前期望大家已经了解基础的 dotnet C# 基础知识，了解基础的概念和项目组织结构

## 项目搭建

本文先从项目搭建开始告诉大家如何创建一个源代码生成器项目。本文后续的内容将会在这个项目中进行演示。本文的编写顺序是先搭建项目，然后再讲解一些基础的概念和用法，再到如何进行调试和分发，最后提供一些实际的演练给到大家

先新建一个控制台项目，新建完成在 Visual Studio 2022 或更高版本中打开项目，双击 csproj 项目文件，即可进行编辑项目文件

本文这里新建了一个名为 `DercelgefarKarhelchaye.Analyzer` 的控制台项目。也许细心的伙伴发现了这个项目使用了 `Analyzer` 作为后缀，这是因为在 dotnet 中源代码生成器和分析器是一体的，按照历史原因的惯性，依然将其命名为分析器项目。在 Visual Studio 2022 的每个项目依赖项里面，大家都会看到一个名为分析器的项，而没有专门一个名为源代码生成器的项，其原因也是如此

如果在这一步就开始卡住了也不用慌，本文在整个过程中都会给出示例代码。我整个代码仓库比较庞大，使用本文各个部分提供的拉取源代码的命令行代码，可以减少拉取的数据，提升拉取的速度，且能够确保切换到正确的 commit 代码

编辑名为 `DercelgefarKarhelchaye.Analyzer` 的控制台项目的 csproj 项目文件，将其 TargetFramework 降级到 netstandard2.0 版本，且按照 dotnet 的惯例，使用 NuGet 添加必要的组件。编辑之后的 csproj 项目文件的内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <LangVersion>latest</LangVersion>
    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.11.0" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.12.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

为什么需要降级为 netstandard2.0 版本？这是为了让此分析器项目能够同时在 dotnet CLI 和 Visual Studio 2022 里面功能。在 Visual Studio 2022 里，当前依然使用的是 .NET Framework 的版本。于是求最小公倍数，选择了 netstandard2.0 版本

以上的 `<LangVersion>latest</LangVersion>` 只是为了方便让咱使用最新的语言特性。前面选择的 netstandard2.0 会导致语言特性默认开得比较低，这里设置为 latest 可以让我们使用最新的语言特性，让代码编写更加方便。这里需要再次提醒，在 dotnet 里面，语言和框架是分开的。使用低版本框架也能使用高版本语言。如果对语言和框架的关系依然有所疑惑，推荐先了解一下 dotnet 的基础知识，不要着急往下看。编写源代码生成器和分析器需要对 dotnet 有一定的了解，否则写着就开始混淆概念了

以上的 `<EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>` 的作用是强制执行扩展分析器规则。这个属性是为了让我们在编写分析器的时候能够更加严格，让我们的代码更加规范。这里大家不需要细致了解，如有兴趣，请参阅 [Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用](https://blog.lindexi.com/post/Roslyn-%E5%88%86%E6%9E%90%E5%99%A8-EnforceExtendedAnalyzerRules-%E5%B1%9E%E6%80%A7%E7%9A%84%E4%BD%9C%E7%94%A8.html )

以上的 `Microsoft.CodeAnalysis.Analyzers` 和 `Microsoft.CodeAnalysis.CSharp` 是必须的组件。`Microsoft.CodeAnalysis.Analyzers` 是分析器的基础组件，`Microsoft.CodeAnalysis.CSharp` 是 C# 的基础组件。这两个组件是必须的，没有这两个组件，我们就无法编写分析器和源代码生成器

通过以上的步骤也可以让大家看到，其实 dotnet 分析器项目也没什么特殊的，依然可以通过一个简单的控制台项目修改而来。其核心关键仅仅只是安装了 `Microsoft.CodeAnalysis.Analyzers` 和 `Microsoft.CodeAnalysis.CSharp` 两个组件而已

现在只是有了一个空的分析器项目，但是还不知道这个项目的效果。为了让分析器项目工作，那就需要有一个被分析的项目。为此咱就再次新建一个控制台项目，让这个控制台项目成为被分析项目

我这里新建了一个名为 `DercelgefarKarhelchaye` 的控制台项目。编辑 `DercelgefarKarhelchaye` 的 csproj 项目文件，让其引用 `DercelgefarKarhelchaye.Analyzer` 项目，且设置 `DercelgefarKarhelchaye.Analyzer` 为分析器。编辑之后的 csproj 项目文件的内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\DercelgefarKarhelchaye.Analyzer\DercelgefarKarhelchaye.Analyzer.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false"/>
  </ItemGroup>

</Project>
```

可以看到以上的 csproj 项目文件和正常的控制台项目的差别仅仅只有在对 `DercelgefarKarhelchaye.Analyzer.csproj` 的引用上。且和正常的引用项目的方式不同的是，这里额外添加了 `OutputItemType="Analyzer" ReferenceOutputAssembly="false"` 两个配置

以上的 `OutputItemType="Analyzer"` 是告诉 dotnet 这个项目是一个分析器项目。这个配置是必须的，没有这个配置，dotnet 就不知道这个项目是一个分析器项目。这个配置是告诉 dotnet 这个项目是一个分析器项目，让 dotnet 在编译的时候能够正确的处理这个项目

以上的 `ReferenceOutputAssembly="false"` 是告诉 dotnet 不要引用这个项目的输出程序集。正常的项目是不应该引用分析器项目的程序集的，分析器项目的作用仅仅只是作为分析器，而不是提供程序集给其他项目引用。这个配置是为了让 dotnet 在编译的时候不要引用这个项目的输出程序集，避免引用错误或导致不小心用了不应该使用的类型

对于正常的项目引用来说，一旦存在项目引用，那被引用的项目的输出程序集就会被引用。此时项目上就可以使用被引用项目的公开类型，以及获取 NuGet 包依赖传递等。但是对于分析器项目来说，这些都是不应该的，正常就不能让项目引用分析器项目的输出程序集。这就是为什么会额外添加 `ReferenceOutputAssembly="false"` 配置的原因

在这里，咱接触到了非常多次的 csproj 项目文件，如果大家对 csproj 项目文件格式感兴趣，请参阅 [理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://blog.walterlv.com/post/understand-the-csproj )

以上的步骤完成之后，最简单的分析器项目和被分析的项目就搭建完成了。这也是分析器的基础，大部分的带分析器的代码都是如此方式搭建的。但也有其他部分是通过 NuGet 带出去的分析器，被 NuGet 带出去的分析器能够更好做到开箱即用，不需要让分析器尝试构建。在后文将会讲解如何将分析器通过 NuGet 带出去，即如何进行分发分析器

现在的分析器项目还没有任何源代码生成和分析的功能，接下来咱将编写简单的源代码生成的代码，让大家看到源代码生成器的效果

## 编写源代码生成器

在 `DercelgefarKarhelchaye.Analyzer` 项目中新建一个名为 `IncrementalGenerator` 的源代码生成器类。编辑 `IncrementalGenerator` 类，让其继承 `IIncrementalGenerator` 接口，实现 `Initialize` 方法，且标记 `[Generator(LanguageNames.CSharp)]` 特性。编辑之后的 `HelloWorldGenerator` 类的内容如下

```csharp
using Microsoft.CodeAnalysis;

namespace DercelgefarKarhelchaye.Analyzer;

[Generator(LanguageNames.CSharp)]
public class IncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        ... // 忽略其他代码
    }
}
```

本文这里直接就是和大家介绍 IIncrementalGenerator 增量 Source Generator 源代码生成器技术，不再介绍 ISourceGenerator 源代码生成器技术。其原因是在 2022 之后，官方大力推荐的是使用 IIncrementalGenerator 增量源代码生成器技术。从业务上讲，仅仅只是 IIncrementalGenerator 多了增量的功能，在进行源代码生成逻辑处理中没有太大的差别。但是在性能上，IIncrementalGenerator 要比 ISourceGenerator 更加高效，更加快速，更加能够防止原本已经很卡的 Visual Studio 更加卡

整个 IIncrementalGenerator 的入口都在 Initialize 方法里面，从 IncrementalGeneratorInitializationContext 参数里可以点出来非常多有用的方法。咱这里先不展开讲解这些方法，先让大家看到一个简单的源代码生成器的效果

在 Initialize 方法里面，咱可以通过 `context.RegisterPostInitializationOutput` 方法注册一个源代码输出。如以下代码所示，将输出一个名为 `GeneratedCode` 的代码

```csharp
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        context.RegisterPostInitializationOutput(initializationContext =>
        {
            initializationContext.AddSource("GeneratedCode.cs",
                """
                using System;
                namespace DercelgefarKarhelchaye
                {
                    public static class GeneratedCode
                    {
                        public static void Print()
                        {
                            Console.WriteLine("Hello from generated code!");
                        }
                    }
                }
                """);
        });
    }
```

预期此时能够将生成的 `GeneratedCode` 类型注入到被分析的项目中。在被分析的项目中，可以通过 `GeneratedCode.Print()` 方法输出 `Hello from generated code!` 字符串

好的，进入到 `DercelgefarKarhelchaye` 项目中，编辑 `Program` 类，调用 `GeneratedCode.Print()` 方法。编辑之后的 `Program` 类的内容如下

```csharp
using DercelgefarKarhelchaye;

GeneratedCode.Print();
```

尝试运行一下 `DercelgefarKarhelchaye` 项目，可以看到控制台输出了 `Hello from generated code!` 字符串。这就是源代码生成器的效果，通过源代码生成器生成的代码，注入到被分析的项目中，让被分析的项目能够使用生成的代码

如此证明了在 `DercelgefarKarhelchaye.Analyzer` 分析器项目中编写的源代码生成器生效了。这就是源代码生成器的基硋，通过源代码生成器生成的代码，注入到被分析的项目中，让被分析的项目能够使用生成的代码

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/95c14524130238b2d6fbca97ca35b89dc921536b/Roslyn/DercelgefarKarhelchaye) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/95c14524130238b2d6fbca97ca35b89dc921536b/Roslyn/DercelgefarKarhelchaye) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 95c14524130238b2d6fbca97ca35b89dc921536b
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 95c14524130238b2d6fbca97ca35b89dc921536b
```

获取代码之后，进入 Roslyn/DercelgefarKarhelchaye 文件夹，即可获取到源代码



- ForAttributeWithMetadataName
- 过于限制，换成直接的分析逻辑。逻辑太复杂了，需要方便调试。不想写单元测试，想直接对项目进行调试


- 编写单元测试的方法
- 日常调试的方法 IsRoslynComponent

- 如何打包 NuGet 包

- 使用视觉辅助了解语法

- 演练 使用 Interceptor 的技术
- 演练 将构建时间写入源代码
- 演练 禁用API调用 分析器

- 源代码生成技术实现中文编程语言 生成的源代码保存到本地文件

- 常用的方法
- 获取配置
- 获取文件的实际本地路径 compilation.Options.SourceReferenceResolver 0b5550ce0e2736df6f2aac01f1f65ca37103fbdf Workbench\KonanohallreGonurliyage




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
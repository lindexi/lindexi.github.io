
# dotnet 源代码生成器分析器入门

本文将带领大家入门 dotnet 的 SourceGenerator 源代码生成器技术，期待大家阅读完本文能够看懂理解和编写源代码生成器和分析器

<!--more-->


<!-- CreateTime:2025/02/11 07:29:34 -->

<!-- 草稿 -->

恭喜你看到了本文，进入到 C# dotnet 的深水区。如果你还是在浅水玩耍的小鲜肉，推荐你点击右上方的关闭按钮，避免受到过于深入的知识的污染

在开始之前期望大家已经了解基础的 dotnet C# 基础知识，了解基础的概念和项目组织结构。本文将尽量使用比较缓的知识爬坡方式编写，以便让大家更舒适地进入到源代码生成器和分析器的世界

在阅读本文过程中，发现本文有任何错误或不足之处，欢迎大家在评论区留言或发送邮件给我，我会尽快修正。如果大家有任何问题或疑问，也欢迎大家在评论区留言或发送邮件给我，我会尽快回复

## 项目搭建

本文先从项目搭建开始告诉大家如何创建一个源代码生成器项目。本文后续的内容将会在这个项目中进行演示。本文的编写顺序是先搭建项目，然后再讲解一些基础的概念和用法，再到如何进行调试和分发，最后提供一些实际的演练给到大家

本文的推荐打开方式是一边阅读本文，一边打开 Visual Studio 2022 或更高版本，对照本文的内容进行操作。照着本文的内容对照着编写代码，可以让大家更好地理解本文的内容，照着过一遍预计就能掌握基础的源代码生成器和分析器的知识，入门源代码生成器和分析器的编写

本文过程中会添加一些外部链接文档，这些外部链接文档都是可选阅读内容，只供大家感兴趣时扩展阅读。本文的核心内容是在本文中编写的，不需要阅读外部链接文档也能够掌握本文的内容

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

为什么需要降级为 netstandard2.0 版本？这是为了让此分析器项目能够同时在 dotnet CLI 和 Visual Studio 2022 里面功能。在 Visual Studio 2022 里，当前依然使用的是 .NET Framework 的版本。于是求最小公倍数，选择了 netstandard2.0 版本。预计后续版本才能使用到最新的 dotnet 框架版本

以上的 `<LangVersion>latest</LangVersion>` 只是为了方便让咱使用最新的语言特性。前面选择的 netstandard2.0 会导致语言特性默认开得比较低，这里设置为 latest 可以让我们使用最新的语言特性，让代码编写更加方便。这里需要再次提醒，在 dotnet 里面，语言和框架是分开的。使用低版本框架也能使用高版本语言。如果对语言和框架的关系依然有所疑惑，推荐先了解一下 dotnet 的基础知识，不要着急往下看。编写源代码生成器和分析器需要对 dotnet 有一定的了解，否则写着就开始混淆概念了

以上的 `<EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>` 的作用是强制执行扩展分析器规则。这个属性是为了让我们在编写分析器的时候能够更加严格，让我们的代码更加规范。这里大家不需要细致了解，如有兴趣，请参阅 [Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用](https://blog.lindexi.com/post/Roslyn-%E5%88%86%E6%9E%90%E5%99%A8-EnforceExtendedAnalyzerRules-%E5%B1%9E%E6%80%A7%E7%9A%84%E4%BD%9C%E7%94%A8.html )
<!-- [Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/17678673.html ) -->

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

可以看到以上的 csproj 项目文件和正常的控制台项目的差别仅仅只有在对 `DercelgefarKarhelchaye.Analyzer.csproj` 的引用上。且和正常的引用项目的方式不同的是，这里额外添加了 `OutputItemType="Analyzer" ReferenceOutputAssembly="false"` 两个配置。这两个配置的作用如下：

- 以上的 `OutputItemType="Analyzer"` 是告诉 dotnet 这个项目是一个分析器项目。这个配置是必须的，没有这个配置，dotnet 就不知道这个项目是一个分析器项目。这个配置是告诉 dotnet 这个项目是一个分析器项目，让 dotnet 在编译的时候能够正确的处理这个项目
- 以上的 `ReferenceOutputAssembly="false"` 是告诉 dotnet 不要引用这个项目的输出程序集。正常的项目是不应该引用分析器项目的程序集的，分析器项目的作用仅仅只是作为分析器，而不是提供程序集给其他项目引用。这个配置是为了让 dotnet 在编译的时候不要引用这个项目的输出程序集，避免引用错误或导致不小心用了不应该使用的类型

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

## 分析和生成入门

在上文中，和大家介绍了如何生成静态的固定的代码内容。在 `RegisterPostInitializationOutput` 方法里面，只允许传递静态固定的代码，不能依据当前项目状态或配置进行动态生成代码。这是因为 `RegisterPostInitializationOutput` 方法的定义上就是用于提供分析器开始分析工作之前的初始化代码。这部分代码由于可不用运行分析过程，可以非常快给到 IDE 层，一般用于提供一些类型定义，可以给到开发者直接快速使用，而不会在使用过程中飘红

上文的代码只是让大家粗略熟悉了一下 `IIncrementalGenerator` 的 API 调用方法。接下来我将带大家开始入门分析器的分析和生成功能

分析和生成很多时候都是不分离的，生成的代码需要依赖分析的结果。为了能让大家更好理解分析器的入门知识，我尝试布置一个任务，接下来让咱根据布置的任务来入门分析和生成功能

### 任务

咱来实现一个经典的需求任务，将项目里面的标记了某个 Attribute 特性的类型全收集起来，最后生成一个代码，让生成的代码输出有哪些类型标记了这个 Attribute 特性，将这些类型的名称输出到控制台

进一步分解任务需求，咱需要有一个源代码生成器。源代码生成器生成两部分代码，第一部分就是 FooAttribute 特性，第二部分就是收集所有标记了 FooAttribute 特性的类型，生成将这些类型的名称输出到控制台的代码。要求全程没有反射参与，全程都是通过 Roslyn 分析和生成完成

### 使用 ForAttributeWithMetadataName 快速分析代码

从工程上进行分析发现，非常大量的分析生成任务都有一个特点，这个特点就是需要找到标记了某个 Attribute 特性的类型或方法或属性等，然后再做某个事情。这个特点其实源自于 dotnet C# 对于 Attribute 特性的设计。Attribute 特性是一种元数据，可以标记在类型、方法、属性等上面，用于描述这个类型、方法、属性等的特性。也常常用于标记给 IDE 和编译器看的，用于告诉 IDE 和编译器这个类型、方法、属性等的特性。比如常用的 `ObsoleteAttribute` 、`CallerMemberNameAttribute` 、`DebuggerDisplayAttribute` 等等

在 `IIncrementalGenerator` 增量 Source Generator 源代码生成器中，提供了 ForAttributeWithMetadataName 工具方法。如此方法名所述，这个方法是用于找到标记了某个 Attribute 特性的类型、方法、属性等。这个方法的使用非常简单，只需要传递一个 Attribute 特性的完整名称，就可以找到标记了这个 Attribute 特性的类型、方法、属性等

在上文的任务中，咱需要找到标记了某个 Attribute 特性的类型，然后将这些类型的名称输出到控制台。这个任务非常适合使用 ForAttributeWithMetadataName 方法来实现。接下来咱就来实现这个任务

依然是新建两个项目，其中一个作为分析器项目，另一个作为被分析的项目。大家既可以在上文现有的项目中继续编写，也可以新建两个项目。这里我新建了一个名为 `NinahajawhuLairfoheahurcee.Analyzer` 的分析器项目，和一个名为 `NinahajawhuLairfoheahurcee` 的被分析项目。本文内容里面只给出关键代码片段，如需要全部的项目文件，可在下文找到所有代码的下载方法。如果自己编写的代码构建不通过或运行输出不符合预期，也推荐大家拉取本文的代码进行阅读

先来完成任务需求分解中的第一部分，编写 FooAttribute 特性代码的生成。由于 FooAttribute 特性的代码不依赖任何分析结果，因此可以使用 RegisterPostInitializationOutput 方法生成。修改上文的 RegisterPostInitializationOutput 注册 `GeneratedCode.cs` 的代码，将其替换为 `FooAttribute.cs` 的生成代码，如下所示

```csharp
[Generator(LanguageNames.CSharp)]
public class IncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        // 先注册一个特性给到业务方使用
        context.RegisterPostInitializationOutput(initializationContext =>
        {
            initializationContext.AddSource("FooAttribute.cs",
                """
                namespace Lindexi;

                public class FooAttribute : Attribute
                {
                }
                """);
        });
    }
}
```

完成这一步之后，即可在业务端编写类型，将类型标记上 FooAttribute 特性。回到名为 `NinahajawhuLairfoheahurcee` 的被分析项目，在 `NinahajawhuLairfoheahurcee` 控制台项目里面添加两个类型，让这两个类型标记上 FooAttribute 特性，用于后续测试类型被收集

```csharp
using Lindexi;

namespace NinahajawhuLairfoheahurcee;

[Foo]
public class F1
{
}

[Foo]
public class F2
{
}
```

在 IIncrementalGenerator 增量 Source Generator 源代码生成，可在 IncrementalGeneratorInitializationContext 里面的 SyntaxProvider 属性，通过 ForAttributeWithMetadataName 快速收集标记了某个特性的类型、属性、方法等等

基本写法格式如下

```csharp
var provider =
    context.SyntaxProvider.ForAttributeWithMetadataName
    (
        "特性名",
        (SyntaxNode node, CancellationToken token) => 语法判断条件,
        (GeneratorAttributeSyntaxContext syntaxContext, CancellationToken token) => 语义处理和获取返回值
    );
```

第一个参数是特性名，记得带上特性的命名空间，以及写明特性的全名。在正常的 C# 代码里面，都会忽略 Attribute 后缀，但是在这里需要带上 Attribute 后缀。第二个参数是语法判断条件，用于判断当前节点是否符合条件。第三个参数是语义处理和获取返回值，用于处理当前节点的语义，获取返回值

那什么是语法，什么是语义呢？ 在 Roslyn 里面，将初步的代码分析的语法层面内容称为 Syntax 语法。语法是非常贴近编写出来的代码直接的内存映射的样子，这个过程里面只做片面考虑，即不考虑代码之间的引用关系，只考虑代码语法本身。语法分析过程是最早的过程，也是损耗极小的过程，也是可以并行化执行的过程。一般来说，进行语法分析都可以将写出来的代码分为一个个 SyntaxTree 语法树，每个代码或代码片都可以转换为一个 SyntaxNode 语法节点

对应于 Syntax 语法的概念，语义 Semantic 则是包含了代码的含义，不仅仅只是语法层面上，语义 Semantic 包含了代码之间的引用关系，包含了各个符号的信息。语义分析过程是在语法分析之后的过程，执行过程中有所损耗，且存在多个代码文件和程序集之间的引用关联关系，这就是为什么在 IIncrementalGenerator 增量 Source Generator 源代码生成设计中是先做语法分析，判断结果通过，再做语义分析的原因

再简单理解可以是如 C# 里面有分部类的概念，进行语法分析的时候，只能一次一个文件一个文件的分析，难以或无法直接分部类的其他分部在哪。但是进行语义分析的时候，可以将所有分部类的信息都收集起来，然后再进行分析，这样就能够找到所有分部类的信息。且在语义分析过程中，能够非常明确知道某个符号的确切含义

语法和语义有比较庞大的知识，我将在后文的专门章节里面详细介绍。这里只是让大家粗略了解一下语法和语义的概念，以便大家能够更好理解后续的内容。本章内容也不会涉及多少的语法和语义知识，不需要对语法和语义有太多的了解，只需要知道这两个概念的存在即可

粗略了解了一点语法和语义的概念，接下来咱就来实现 ForAttributeWithMetadataName 方法的使用。在 `NinahajawhuLairfoheahurcee.Analyzer` 分析器项目中，修改 IncrementalGenerator 类的 Initialize 方法，添加 ForAttributeWithMetadataName 方法的使用，如下所示

```csharp
        IncrementalValuesProvider<string> targetClassNameProvider = context.SyntaxProvider.ForAttributeWithMetadataName("Lindexi.FooAttribute",
            // 进一步判断
            (SyntaxNode node, CancellationToken token) => node.IsKind(SyntaxKind.ClassDeclaration),
            (GeneratorAttributeSyntaxContext syntaxContext, CancellationToken token) => syntaxContext.TargetSymbol.Name);
```

如上面代码所示，第一个参数传入特性名，即 `"Lindexi.FooAttribute"` 字符串。此时将进入预设逻辑，增量的寻找所有标记了名为 `"Lindexi.FooAttribute"` 特性的类型或属性或方法等等代码。一旦找到了标记了 `"Lindexi.FooAttribute"` 特性的代码，将会进入第二个参数的语法判断条件，即 `(SyntaxNode node, CancellationToken token) => node.IsKind(SyntaxKind.ClassDeclaration)` 代码块。此时将进入进一步判断，只有当找到的代码是类声明的时候，才是符合咱的任务需求的代码，即满足感兴趣的条件。这里的 SyntaxNode.IsKind 方法是判断当前传入的 SyntaxNode 是什么。前面步骤只是找到了标记了 `"Lindexi.FooAttribute"` 特性的代码，这里进一步判断找到的代码是不是类声明。满足前两个步骤，则证明这是一个在类型上面标记了名为 `"Lindexi.FooAttribute"` 特性的代码，可以进入最后一个参数里面进行进一步的语义处理

进一步的语义处理是 `(GeneratorAttributeSyntaxContext syntaxContext, CancellationToken token) => syntaxContext.TargetSymbol.Name` 代码块。这里的 GeneratorAttributeSyntaxContext.TargetSymbol 属性是当前找到的代码的符号，即当前找到的代码的语义信息。这里的 TargetSymbol.Name 属性是当前找到的代码的名称，即当前找到的代码的类型名称。这里的代码块返回的是当前找到的代码的类型名称，即当前找到的代码的名称

将其返回的内容是类似 Linq 的查询结果，即 `IncrementalValuesProvider<string>` 类型。这个类型是一个增量的值提供者，而不是立刻就返回一次所有满足条件的代码。在 Visual Studio 里面的执行逻辑上，大家可以认为是每更改、新增一次代码，就会执行一次这个查询逻辑，整个查询逻辑是源源不断执行的，不是一次性的，也不是瞬时全跑的，而是增量的逐步执行的

执行过程也是一级级执行的，先通过了第一个参数的特性名，快速判断是否满足参数条件，再经过第二个参数进行语法判断。经过前面两个参数判断就可以快速过滤掉大量的代码，如此的方式可以极大减少计算工作量

现在拿到了 `IncrementalValuesProvider<string>` 返回值，能够从这里源源不断取出一个个类型出来。但按照咱的任务需求，咱是需要一口气收集所有类型的，不能一个个慢慢取。为此咱需要将 `IncrementalValuesProvider<string>` 给收集起来，成为一个集合数组。这里可以使用 Collect 方法进行收集，如下所示

```csharp
        IncrementalValueProvider<ImmutableArray<string>> targetClassNameArrayProvider = targetClassNameProvider
            .Collect();
```

可以看到此时返回值就从 `IncrementalValuesProvider<string>` 类型转换为 `IncrementalValueProvider<ImmutableArray<string>>` 类型。核心不同在于 `string` 和 `ImmutableArray<string>` 不可变数组的差异而已。在整个 Roslyn 设计里面，大量采用不可变思想，这里的返回值就是不可变思想的一个体现

最后一步就是将 `IncrementalValueProvider<ImmutableArray<string>>` 返回值注册到输出源代码中。在 `IncrementalGenerator` 类的 Initialize 方法里面，使用 `context.RegisterSourceOutput` 方法注册输出源代码，如下所示

```csharp
        context.RegisterSourceOutput(targetClassNameArrayProvider, (productionContext, classNameArray) =>
        {
            productionContext.AddSource("GeneratedCode.cs",
                $$"""
                using System;
                namespace NinahajawhuLairfoheahurcee
                {
                    public static class GeneratedCode
                    {
                        public static void Print()
                        {
                            Console.WriteLine("标记了 Foo 特性的类型有： ｛｛string.Join(",", classNameArray)｝｝");
                        }
                    }
                }
                """);
        });
```

尝试运行控制台项目，可见此时能够输出以下内容到控制台

```
标记了 Foo 特性的类型有： F1,F2
```

尝试展开 Visual Studio 的 依赖项->分析器，如下图所示

<!-- ![](image/dotnet 源代码生成器分析器入门/dotnet 源代码生成器分析器入门0.png) -->
![](http://cdn.lindexi.site/lindexi%2F20252142036427475.jpg)

可以看到生成的代码如下

```csharp
using System;
namespace NinahajawhuLairfoheahurcee
{
    public static class GeneratedCode
    {
        public static void Print()
        {
            Console.WriteLine("标记了 Foo 特性的类型有： F2,F1");
        }
    }
}
```

如此以来，对比传统的反射的方法，源代码生成的方式可以将耗时完全放在开发编译过程，不会占用用户端的执行时间。且这个过程都是完完全全的直接代码，也方便运行时的 JIT 进行优化，大大提升了运行时间。完完全全的直接代码也带来了静态分析的友好，可以作为代码裁剪和 AOT 的底层支持

喜欢点点的伙伴也许在准备写 RegisterSourceOutput 的时候，就发现了还有一个名为 RegisterImplementationSourceOutput 方法，那 RegisterSourceOutput 和 RegisterImplementationSourceOutput 的差别是什么？这两个方法对最终生成的代码是没有影响的，核心差别是 RegisterImplementationSourceOutput 是用来注册具体实现生成的代码，这部分输入的代码会被 IDE 作为可选分析项。如 RegisterImplementationSourceOutput 命名所述，这是一个用来注册“具体实现”的代码，在代码里面，咱可以强行将代码分为“定义代码”和“实现代码”，比如说方法签名是定义代码，方法体是实现代码。从 IDE 的分析角度来看，只对“定义代码”而跳过“实现代码”，可以更大程度的减少分析压力，提升分析速度。通过 RegisterImplementationSourceOutput 方法注册的代码，会被 IDE 作为可选分析项，不会因为生成了大量代码导致 IDE 过于卡顿。但带来的问题是这部分生成代码可能不被加入 IDE 分析，导致业务方调用时飘红。因此通过 RegisterImplementationSourceOutput 生成的代码，基本要求是不会被业务方直接调用。常用的套路是先通过 RegisterSourceOutput 或甚至是 RegisterPostInitializationOutput 生成分部类或分部方法，然后再慢慢在 RegisterImplementationSourceOutput 里面填充实现代码

以上就是通过 ForAttributeWithMetadataName 开始入门编写分析和收集和生成的简单例子，如果对 ForAttributeWithMetadataName 使用方法感兴趣，请参阅 [使用 ForAttributeWithMetadataName 提高 IIncrementalGenerator 增量 Source Generator 源代码生成开发效率和性能](https://blog.lindexi.com/post/%E4%BD%BF%E7%94%A8-ForAttributeWithMetadataName-%E6%8F%90%E9%AB%98-IIncrementalGenerator-%E5%A2%9E%E9%87%8F-Source-Generator-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%BC%80%E5%8F%91%E6%95%88%E7%8E%87%E5%92%8C%E6%80%A7%E8%83%BD.html ) 
<!-- [使用 ForAttributeWithMetadataName 提高 IIncrementalGenerator 增量 Source Generator 源代码生成开发效率和性能 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18009107 ) -->

如果大家照着以上的例子编写不出来能构建通过的代码，或者是运行代码不符合预期，欢迎拉取我的示例代码进行阅读

同样的，以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b8c036de9d9d7c4b1a3d329054086d6566d14dc4/Roslyn/NinahajawhuLairfoheahurcee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b8c036de9d9d7c4b1a3d329054086d6566d14dc4/Roslyn/NinahajawhuLairfoheahurcee) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b8c036de9d9d7c4b1a3d329054086d6566d14dc4
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b8c036de9d9d7c4b1a3d329054086d6566d14dc4
```

获取代码之后，进入 Roslyn/NinahajawhuLairfoheahurcee 文件夹，即可获取到源代码

## 更底层的收集分析和生成

阅读到这里，也许大家会感慨，使用 `ForAttributeWithMetadataName` 还是有很大的限制。比如我的需求任务是分析任意的继承了 IFoo 接口的代码，而没有任何的标记，那应该如何做呢？只通过 `ForAttributeWithMetadataName` 是无法实现的。这个时候就需要更底层的收集分析和生成技术

本文会和大家介绍 `ForAttributeWithMetadataName` 仅仅只是因为 `ForAttributeWithMetadataName` 方法使用简单，且使用频率高。不代表只能通过 `ForAttributeWithMetadataName` 方法进行分析和生成。实际上，`ForAttributeWithMetadataName` 方法只是对更底层的收集分析和生成技术的封装，更底层的收集分析和生成技术是可以实现更多的需求任务的

在 IIncrementalGenerator 增量 Source Generator 源代码生成里面提供了众多数据源入口，比如整个的配置、引用的程序集、源代码等等。最核心也是用最多的就是通过提供的源代码数据源进行收集分析

按照官方的设计，将会分为三个步骤完成增量代码生成：

1. 告诉框架层需要关注哪些文件或内容或配置的变更
  - 在有对应的文件等的变更情况下，才会触发后续步骤。如此就是增量代码生成的关键
2. 告诉框架层从变更的文件里面感兴趣什么数据，对数据预先进行处理
  - 预先处理过程中，是会不断进行过滤处理的，确保只有感兴趣的数据才会进入后续步骤
  - 其中第一步和第二步可以合在一起
3. 使用给出的数据进行处理源代码生成逻辑
  - 这一步的逻辑和普通的 Source Generator 是相同的，只是输入的参数不同







可以看到在 `IIncrementalGenerator` 这部分设计里面是非常靠近 Linq 的设计的。这更底层的设计上，所期望的就是让数据可以和 Linq 的数据流设计一样，能够一级级传递，且过程中是 Lazy 的和带缓存的。核心目的就是减少计算压力，充分利用 Roslyn 的不可变性带来的缓存机制，减少分析过程的计算压力，不让原本就很卡的 Visual Studio 更加卡


- 过于限制，换成直接的分析逻辑。逻辑太复杂了，需要方便调试。不想写单元测试，想直接对项目进行调试

### 编写单元测试

### 直接调试项目

## 使用语法可视化窗格辅助了解语法

语法还是太复杂了，不知道怎么写。这个时候可以使用视觉辅助了解语法

[Roslyn 入门：使用 Visual Studio 的语法可视化（Syntax Visualizer）窗格查看和了解代码的语法树 - walterlv](https://blog.walterlv.com/post/roslyn-syntax-visualizer )


学习了这么多，可以试试进行一些实践演练

是否源代码生成器能够干的话，程序猿也能干？介绍 Interceptor 技术


- 编写单元测试的方法
- 日常调试的方法 IsRoslynComponent

- 如何打包 NuGet 包

- 使用视觉辅助了解语法

- 演练 使用 Interceptor 的技术
- 演练 将构建时间写入源代码
- 从文件写入代码
- 演练 禁用API调用 分析器

- 源代码生成技术实现中文编程语言 生成的源代码保存到本地文件

- 常用的方法
- 获取配置
- 获取文件的实际本地路径 compilation.Options.SourceReferenceResolver 0b5550ce0e2736df6f2aac01f1f65ca37103fbdf Workbench\KonanohallreGonurliyage

[使用 Roslyn 分析代码注释，给 TODO 类型的注释添加负责人、截止日期和 issue 链接跟踪 - walterlv](https://blog.walterlv.com/post/comment-analyzer-and-code-fix-using-roslyn.html )

[Roslyn 入门：使用 Roslyn 静态分析现有项目中的代码（语法分析） - walterlv](https://blog.walterlv.com/post/analysis-code-of-existed-projects-using-roslyn.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
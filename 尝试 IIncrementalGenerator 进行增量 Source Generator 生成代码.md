# 尝试 IIncrementalGenerator 进行增量 Source Generator 生成代码

在加上热重载时，源代码生成 Source Generator 的默认行为会让 Visual Studio 有些为难，其原因是热重载会变更代码，变更代码触发代码生成器更新代码，代码生成器更新的代码说不定又会有某些逗比逻辑再次触发热重载。于是就会发现在某些复杂的项目下，开启热重载之后，在编辑并继续界面将会等非常久，甚至再也无法继续。为了解决这个问题，大聪明设计了 Incremental Generators 机制，此 Incremental Generators 机制和 Source Generator 不冲突，被设计用来解决热重载的源代码生成性能问题，本文将告诉大家此新的 API 的入门级使用

<!--more-->
<!-- CreateTime:2022/8/29 8:40:56 -->

<!-- 发布 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

开始阅读之前必须要明确的是，几乎所有的设计为高性能使用的 API 都代表着 API 本身不够好用。本身使用 Source Generator 就有一定的门槛了，现在使用 Incremental Generators 机制只会更加复杂。在开始阅读本文之前，我期望你已熟悉源代码生成机制以及 dotnet 的构建过程。本文非新手友好

我开始是不知道还有 Incremental Generators 机制的存在。之所以有了解到这么强大的机制的存在还是在一次我的 Visual Studio 卡炸了，我给 Visual Studio 官方报告了问题，然后 [Sam Harwell](https://github.com/sharwell) 经过了分析，找到了是我所在团队写的 [dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 库的 Source Generator 影响了 Visual Studio 的性能，大佬给的建议是在这个库上 Incremental Generators 机制，详细请看 [https://github.com/dotnet-campus/dotnetCampus.Ipc/issues/97](https://github.com/dotnet-campus/dotnetCampus.Ipc/issues/97)

当前是关于 Incremental Generators 的使用还没有非常正式的文档，官方权威的文档只有放在 GitHub 的文档： [https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.md](https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.md)

这也就是为什么大佬在今年 2022 年 5 月告诉我这个技术，我到现在还没有开始动手的原因。我感觉他有点将我当成小白鼠了，毕竟要是我翻车了，我肯定会去找他。然而他没想到的是，我最近的技能都点去 MAUI 去了，毕竟 Visual Studio 卡的话，那不是日常么。尽管入门的等级我学会了，但感觉改造 [dotnetCampus.Ipc](https://github.com/dotnet-campus/dotnetCampus.Ipc) 还是一个大工程，先写博客再说

尽管文档不多，但是好在还是延续 Roslyn 的设计，看 API 不看文档的情况下，我还是能用起来。只是在刚开始用的时候就被自己逗了一下哈，在熟悉 Source Generator 的基础上，用起来的难度很低。 我看了 GitHub 上的文档，这篇文档很长，前部分在介绍设计和约束，中间是核心介绍，接下来也就是占最多的部分是讲 API 的设计和具体行为，最后是一些优化设计。如果只是入门的话，看中间的核心介绍也就足够了，甚至看介绍的代码即可

为了方便演示，本文将创建一个叫 WhacadenaKewarfellaja 的项目，和一个用来做代码生成的 WhacadenaKewarfellaja.Analyzers 项目。本文的所有代码都可以在本文最后找到下载地址

推荐大家按照本文的步骤，照着做一次，做一次将会更好的了解增量的代码生成的逻辑是如何写的

在 WhacadenaKewarfellaja 项目里面，只包括一个叫 Program.cs 的文件，这个文件的代码如下，只是定义一个主函数和调用一个叫 HelloFrom 的分部函数，此 HelloFrom 分部函数将会由代码生成项目生成实际定义代码

```csharp
namespace WhacadenaKewarfellaja
{
    public static partial class Program
    {
        public static void Main(string[] args)
        {
            HelloFrom("Fxx");

            Console.WriteLine("Hello, World!");
        }

        static partial void HelloFrom(string name);
    }
}
```

在 WhacadenaKewarfellaja 的项目文件 csproj 文件里，加上了对 WhacadenaKewarfellaja.Analyzers 代码生成项目的引用，加上此应用的用途是为了方便调试代码生成项目。通过 [使用 Source Generator 在编译你的 .NET 项目时自动生成代码 - walterlv](http://blog.walterlv.com/post/generate-csharp-source-using-roslyn-source-generator) 可以了解到，要让代码生成项目可以运行，有两个方法，第一个方法就是打包为 NuGet 包，通过引用 NuGet 的方式执行到代码生成项目。第二个方法是通过本文以下使用的项目引用方法，使用项目引用的方法更加方便调试。使用 NuGet 的方法是用来进行发布，两个不冲突

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\WhacadenaKewarfellaja.Analyzers\WhacadenaKewarfellaja.Analyzers.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false"/>
  </ItemGroup>

</Project>
```

从以上代码可以看到，和其他的项目引用唯一的不同点在于 ProjectReference 设置了 `OutputItemType="Analyzer" ReferenceOutputAssembly="false"` 两个属性

如此即可完成搭建被测试的项目，预期是在执行 WhacadenaKewarfellaja 应用，调用的 HelloFrom 方法的实际逻辑是由 WhacadenaKewarfellaja.Analyzers 生成的代码。下一步是搭建 WhacadenaKewarfellaja.Analyzers 的项目

编辑 WhacadenaKewarfellaja.Analyzers 的 csproj 项目文件，替换为以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.3" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.2.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

如 [使用 Source Generator 在编译你的 .NET 项目时自动生成代码 - walterlv](http://blog.walterlv.com/post/generate-csharp-source-using-roslyn-source-generator) 博客提到，此项目要求使用 TargetFramework 为 netstandard2.0 的版本，如此才能有更好的兼容性。核心原因是 Visual Studio 2022 现在还没有能完全迁移到 dotnet core 上，而 dotnet 工具本身是跟随 SDK 走的，两个构建工具有所不同，为了更好的兼容，就期望使用 .NET Standard 2.0 版本

以上代码的 AppendTargetFrameworkToOutputPath 属于可选项，只是为了让输出的文件夹更加清真

上面代码也通过 PackageReference 分别安装了 Microsoft.CodeAnalysis.Analyzers 和 Microsoft.CodeAnalysis.CSharp 库。采用这两个 NuGet 包即可进行源代码生成

在 WhacadenaKewarfellaja.Analyzers 新建一个叫 CodeCollectionIncrementalGenerator 的类型，类型名称随意，将在此类型里面编写增量的代码生成逻辑

增量代码生成和普通的 Source Generator 一样，需要在代码生成入口类型上标记特性，如以下代码

```csharp
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator
    {
    }
```

为了使用增量的代码生成，需要让 CodeCollectionIncrementalGenerator 继承 IIncrementalGenerator 接口，实现 Initialize 方法，代码如下

```csharp
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
        }
    }
```

对于增量的代码生成，只需要有 Initialize 方法即可，所有逻辑都在这个方法里面实现

按照官方的设计，将会分为三个步骤完成增量代码生成：

1. 告诉框架层需要关注哪些文件的变更
  - 在有对应的文件的变更情况下，才会触发后续步骤。如此就是增量代码生成的关键
2. 告诉框架层从变更的文件里面感兴趣什么数据，对数据预先进行处理
  - 预先处理过程中，是会不断进行丢掉处理的
  - 其中第一步和第二步可以合在一起
3. 使用给出的数据进行处理源代码生成逻辑
  - 这一步的逻辑和普通的 Source Generator 是相同的，只是输入的参数不同

下面开始编写代码逻辑

先从 IncrementalGeneratorInitializationContext 里面获取参数，例如对全体的尝试构建的文件感兴趣，可以使用如下代码

```csharp
context.CompilationProvider
```

又如官方给出的例子，对所有的后缀名为 txt 的附加文件感兴趣的代码可以如此写

```csharp
        IncrementalValuesProvider<AdditionalText> textFiles = initContext.AdditionalTextsProvider.Where(static file => file.Path.EndsWith(".txt"));
```

也就是说对于增量代码生成来说，可以很方便的实现从其他文件生成源代码的功能。例如通过 xml 文件生成代码等。这就是为什么 WPF 仓库里面，有大佬在尝试使用源代码生成的方式实现 XAML 生成 cs 代码的原因

继续回到本文的例子，本文的逻辑是对任何的尝试构建的代码变更感兴趣，变更之后获取的是整个代码的信息。当然，这只是本文的例子，在很多情况下，都会附加很多判断逻辑

```csharp
            // 找到对什么文件感兴趣
            IncrementalValueProvider<Compilation> compilations =
                context.CompilationProvider
                        // 这里的 Select 是仿照 Linq 写的，可不是真的 Linq 哦，只是一个叫 Select 的方法
                        // public static IncrementalValueProvider<TResult> Select<TSource,TResult>(this IncrementalValueProvider<TSource> source, Func<TSource,CancellationToken,TResult> selector)
                    .Select((compilation, cancellationToken) => compilation);
```

这里的 Select 等方法是仿照 Linq 写的，可不是真的 Linq 哦，只是一个叫 Select 的方法，方法的定义如下

```csharp
 public static IncrementalValueProvider<TResult> Select<TSource,TResult>(this IncrementalValueProvider<TSource> source, Func<TSource,CancellationToken,TResult> selector)
```

我就是在这里被逗了，以为是通过 Linq 的方式，返回集合。由于是以为返回的是集合，对于增量的理解就错了。这里其实一个写入条件，后续将会根据条件决定是否执行增量的逻辑。例如写了 Where 过滤，如官方例子里面，只是对于 txt 文件的变更感兴趣，那如果改动的是其他的 xml 文件，那自然不会触发后续逻辑

大部分的写法是使用 Where 进行过滤，获取到需要增量感兴趣的变更，接下来通过 Select 进行数据处理。数据处理过程随时可以被打断，因为用户的增量变更可能是用户习惯不断按下 ctrl+s 键保存，如果快速两次保存，第一次就不需要等待执行完成

本文的例子里面是使用所有的代码信息，这是不推荐的，但是其实没啥问题

接下来编辑核心的逻辑，根据增量的信息生成代码

使用 IncrementalGeneratorInitializationContext 的 RegisterSourceOutput 注册源代码生成的逻辑

```csharp
     context.RegisterSourceOutput(compilations, (sourceProductionContext, compilation) =>
     {

     });
```

这里的 RegisterSourceOutput 参数比较复杂，要求注入条件，和执行的委托。注入的条件就是 `IncrementalValueProvider<T>` 类型的参数，如上文，采用 Where 或 Select 配合创建的 compilations 变量。执行的委托有两个参数，分别是源代码生产的上下文信息，可以在此上下文信息里面传入源代码，以及注入的条件过滤之后的内容

从 IncrementalGeneratorInitializationContext 里构建出 `IncrementalValueProvider<T>` 对象，此对象用作过滤条件和获取信息，在 RegisterSourceOutput 里，将 `IncrementalValueProvider<T>` 的 `T` 作为参数传入到委托。底层实现是通过 RegisterSourceOutput 的第一个参数，也就是 `IncrementalValueProvider<T>` 进行过滤当前的增量变更，发现有数据可以输出时，则调用后续传入的委托方法

在委托方法里面，和其他的源代码生成的不同只在于参数。例如本文的代码将获取参数里传入的 compilation 用来判断是否 Program 类型发生变更，如有变更，那么重新生成 HelloFrom 方法

判断传入的 compilation 是否包含 Program 类型的变更，第一步是获取到符号树

```csharp
            context.RegisterSourceOutput(compilations, (sourceProductionContext, compilation) =>
            {
                var syntaxTree = compilation.SyntaxTrees.FirstOrDefault();
                if (syntaxTree == null)
                {
                    return;
                }
            });
```

接着判断是否类型是 Program 类型

```csharp
                var root = syntaxTree.GetRoot(sourceProductionContext.CancellationToken);

                // 选择给 Program 的附加上
                var classDeclarationSyntax = root
                    .DescendantNodes(descendIntoTrivia: true)
                    .OfType<ClassDeclarationSyntax>()
                    .FirstOrDefault();
                if (classDeclarationSyntax?.Identifier.Text!="Program")
                {
                    // 如果变更的非预期类型，那就不加上代码，否则代码将会重复加入
                    return;
                }
```

如果是的话，那就加上生成的源代码

```csharp
                string source = @"
using System;

namespace WhacadenaKewarfellaja
{
    public static partial class Program
    {
        static partial void HelloFrom(string name)
        {
            Console.WriteLine($""Says: Hi from '{name}'"");
        }
    }
}
";
                sourceProductionContext.AddSource("GeneratedSourceTest", source);
```

调用 AddSource 方法将源代码字符串加入即可。尝试运行 WhacadenaKewarfellaja 项目，可以看到调用 HelloFrom 的输出

但是如此写和普通的源代码生成有什么差别？其实最大的不同在于性能上，通过此方法可以使用很多缓存，减少生成的数据。例如可以定义一个静态的属性，通过此静态的属性了解增量的源代码生成被调用的次数。将以上代码的源代码生成 source 字符串的进行更改

```csharp
                // 这是一个很强的技术，在代码没有变更的情况下，多次构建，是可以看到不会重复进入此逻辑，也就是 Count 属性没有加一
                // 可以试试对一个大的项目，修改部分代码，看看 Count 属性

                string source = $@"Console.WriteLine(""构建 {Count} 次"");";
                sourceProductionContext.AddSource("GeneratedSourceTest", source);

                Count++;

        private static int Count { set; get; } = 0;
```

可以试试不断构建代码或者更改代码之后再构建，通过输出了解 Count 的值，从而了解到这个增量生成源代码的性能提升

以下是核心的代码

```csharp
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            // 找到对什么文件感兴趣
            // 例如对全体的 cs 代码感兴趣
            IncrementalValueProvider<Compilation> compilations =
                context.CompilationProvider
                        // 这里的 Select 是仿照 Linq 写的，可不是真的 Linq 哦，只是一个叫 Select 的方法
                        // public static IncrementalValueProvider<TResult> Select<TSource,TResult>(this IncrementalValueProvider<TSource> source, Func<TSource,CancellationToken,TResult> selector)
                    .Select((compilation, cancellationToken) => compilation);

            context.RegisterSourceOutput(compilations, (sourceProductionContext, compilation) =>
            {
                var syntaxTree = compilation.SyntaxTrees.FirstOrDefault();
                if (syntaxTree == null)
                {
                    return;
                }

                var root = syntaxTree.GetRoot(sourceProductionContext.CancellationToken);

                // 选择给 Program 的附加上
                var classDeclarationSyntax = root
                    .DescendantNodes(descendIntoTrivia: true)
                    .OfType<ClassDeclarationSyntax>()
                    .FirstOrDefault();
                if (classDeclarationSyntax?.Identifier.Text!="Program")
                {
                    // 如果变更的非预期类型，那就不加上代码，否则代码将会重复加入
                    return;
                }

                // 这是一个很强的技术，在代码没有变更的情况下，多次构建，是可以看到不会重复进入此逻辑，也就是 Count 属性没有加一
                // 可以试试对一个大的项目，修改部分代码，看看 Count 属性

                string source = ... // 生成的代码
                sourceProductionContext.AddSource("GeneratedSourceTest", source);

                Count++;
            });
        }

        private static int Count { set; get; } = 0;
    }
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/cfa6270bf2695851672ef9bad0deb2ef21f5ba2a/WhacadenaKewarfellaja) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cfa6270bf2695851672ef9bad0deb2ef21f5ba2a/WhacadenaKewarfellaja) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cfa6270bf2695851672ef9bad0deb2ef21f5ba2a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cfa6270bf2695851672ef9bad0deb2ef21f5ba2a
```

获取代码之后，进入 WhacadenaKewarfellaja 文件夹
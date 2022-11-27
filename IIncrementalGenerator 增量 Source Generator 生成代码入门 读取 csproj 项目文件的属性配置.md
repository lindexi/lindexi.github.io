# IIncrementalGenerator 增量 Source Generator 生成代码入门 读取 csproj 项目文件的属性配置

本文告诉大家如何在使用 IIncrementalGenerator 进行增量的 Source Generator 生成代码时，读取项目里的项目文件属性，从而实现为项目定制的逻辑。或者是读取 NuGet 包里面的一些配置，从而方便实现逻辑

<!--more-->
<!-- CreateTime:2022/11/7 8:04:32 -->


<!-- 发布 -->
<!-- 博客 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

使用增量的源代码生成具有更高的门槛。本文属于入门博客，但非编程新手友好，期望阅读本文之前，已了解源代码生成和项目构建和项目组织的基础知识

阅读本文，你可以了解到如何在进行增量的源代码生成过程中，读取项目文件里面的属性，从而执行特殊的逻辑

本文的例子期望达成的是，读取 csproj 项目文件里面的 MyCustomProperty 属性，将此属性的文本内容，作为生成代码的一部分。以下代码是 MyCustomProperty 属性的定义。值得一说的是，此方法不仅仅适合用在读取 csproj 项目文件里面的属性，也适合用来读取 NuGet 包的 xx.props 和 xx.targets 文件里面的属性

```xml
  <PropertyGroup>
    <MyCustomProperty>lindexi is doubi</MyCustomProperty>
  </PropertyGroup>
```

在例子代码里面，期望能够将 MyCustomProperty 属性的内容，作为控制台输出的参数，输出到。相当于将 MyCustomProperty 属性的内容，放入到下面代码的 text 变量里面，加入到源代码生成

```csharp
                    var code = @"using System;
namespace LainewihereJerejawwerye
{
    public static class Foo
    {
        public static void F1()
        {
            Console.WriteLine(""" + text + @""");
        }
    }
}";
```

接下来是开始写这个例子的代码，本文的所有代码都可以在本文末尾找到下载地址

开始之前，按照惯例，先新建两个项目，分别是 LainewihereJerejawwerye 和 LainewihereJerejawwerye.Analyzers 两个项目。其中 LainewihereJerejawwerye 用来安装使用分析器的项目，提供 MyCustomProperty 属性。在 LainewihereJerejawwerye.Analyzers 里面，作为分析器项目，将实现源代码生成逻辑

编辑 LainewihereJerejawwerye.Analyzers 的 csproj 项目文件，替换为以下代码。下面代码的细节请参阅 [使用 Source Generator 在编译你的 .NET 项目时自动生成代码 - walterlv](http://blog.walterlv.com/post/generate-csharp-source-using-roslyn-source-generator) 博客

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

接着编辑 LainewihereJerejawwerye 项目的 csproj 项目文件，让他引用上分析器项目。额外的加上 MyCustomProperty 属性，修改之后的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <PropertyGroup>
    <MyCustomProperty>lindexi is doubi</MyCustomProperty>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\LainewihereJerejawwerye.Analyzers\LainewihereJerejawwerye.Analyzers.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
  </ItemGroup>

</Project>

```

根据官方的 [Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md) 文档，想要让分析器项目能够拿到 csproj 项目文件里面的属性，就需要明确使用 CompilerVisibleProperty 包含其对分析器可见的属性名。在属性系统里面，可以分为全局属性以及单项属性。所谓全局属性，就是对整个项目可用，而不是对项目里的某个文件进行设置的属性。单项属性就是对单个项，如单个文件进行设置的额外的配置属性。本文这里只讨论全局的属性配置情况，也就是对整个项目的配置的属性

如上文描述，添加一个 CompilerVisibleProperty 包含对分析器可见的 MyCustomProperty 属性，代码如下

```xml
  <ItemGroup>
    <CompilerVisibleProperty Include="MyCustomProperty" />
  </ItemGroup>
```

加上 CompilerVisibleProperty 之后，分析器才可以通过 GlobalOptions 获取属性。获取时，需要分析器项目使用 TryGetValue 方法，且要求在属性前面加上 `build_property.` 前缀。下文的例子将会告诉大家具体的获取方法

这里还存在一个问题，那就是属性的时机，如果属性的赋值是在分析器执行完成之后再赋值，那自然会让分析器拿不到符合预期的属性内容。而如果属性过早赋值，可能属性本身的逻辑无法实现。因此需要找到一个最迟的时机，这是在分析器可以获取到属性内容的最后时机，如以下代码，可以放在 GenerateMSBuildEditorConfigFileCore 执行之前

```xml
  <Target Name="Xxxxxxxx" BeforeTargets="GenerateMSBuildEditorConfigFileCore">
    <PropertyGroup>
      <MyCustomProperty>xxxxx</MyCustomProperty>
    </PropertyGroup>
  </Target>
```

如果属性能够一开始就赋值，那推荐就是一开始就赋值。如果属性有其他依赖，那推荐使用类似上面代码的写法。如果属性需要在 GenerateMSBuildEditorConfigFileCore 才获取到内容的，那就凉凉了，需要修改实现

完成配置之后，开始编写分析器项目的代码，由于分析器项目采用的是增量源代码构建，逻辑上会比较复杂一些。在增量源代码生成里面，是没有直接提供 GlobalOptions 用来访问的，而是需要按照增量的方法，先过滤出感兴趣的内容。在感兴趣的内容发生变更或初始化时，将会触发实际执行的逻辑，在实际执行的逻辑，通过过滤条件的输出结果，拿到参数，生成代码

先开始搭建基础的代码，在 LainewihereJerejawwerye.Analyzers 新建一个叫 CodeCollectionIncrementalGenerator 的类型，此类将用来编写本文的核心代码。类名随意，可以自己修改

```csharp
using System;
using Microsoft.CodeAnalysis;

namespace LainewihereJerejawwerye.Analyzers
{
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            // 在这里编写代码
        }
    }
}
```

使用增量代码生成，可以看到继承的是 IIncrementalGenerator 接口，需要实现的只有初始化函数，而不是一个初始化和一个执行函数。在增量代码生成里，需要在此初始化函数里面完成所有代码逻辑。但不代表着就是在初始化函数里面执行完成，因为实际上在此初始化函数里面，更多的是注入各个委托，在各个委托里面实现逻辑。在编写代码过程中，各个委托将会按需被调度执行，从而完成增量代码生成

按照增量代码生成的编写要求，第一步是声明对什么感兴趣，也就是一次过滤。只有满足条件的内容发生变更或初始化时，才会触发后续逻辑，同时过滤的结果也会作为后续逻辑的输入参数。本文这里需要的只是配置属性而已。配置属性都放在 `AnalyzerConfigOptionsProvider` 里，换句话说，我可以对整个 AnalyzerConfigOptionsProvider 都感兴趣。于是 AnalyzerConfigOptionsProvider 属性就是我的过滤条件

于是将 AnalyzerConfigOptionsProvider 作为参数条件传入到 RegisterImplementationSourceOutput 方法里面，也就是只有在配置初始化或变更时，才会触发传入 RegisterImplementationSourceOutput 的委托

```csharp
            context.RegisterImplementationSourceOutput(context.AnalyzerConfigOptionsProvider,
                (productionContext, provider) =>
                {
                    // 这里的代码只有当配置初始化或变更时才会被执行
                };
```

这里拿到的 `provider` 就是项目的配置了，其中本文期望的 csproj 项目文件的属性也就在 GlobalOptions 属性里面，可以通过如下代码进行获取

```csharp
            context.RegisterImplementationSourceOutput(context.AnalyzerConfigOptionsProvider,
                (productionContext, provider) =>
                {
                    var text = string.Empty;

                    // 通过 csproj 等 PropertyGroup 里面获取
                    // 需要将可见的，放入到 CompilerVisibleProperty 里面
                    // 需要加上 `build_property.` 前缀
                    if (provider.GlobalOptions.TryGetValue("build_property.MyCustomProperty", out var myCustomProperty))
                    {
                        text += " " + myCustomProperty;
                    }
                };
```

如此即可拿到属性的内容，放入到 `text` 变量。接着再使用本文已开始的生成代码，完成之后的代码如下

```csharp
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            context.RegisterImplementationSourceOutput(context.AnalyzerConfigOptionsProvider,
                (productionContext, provider) =>
                {
                    var text = string.Empty;

                    // 通过 csproj 等 PropertyGroup 里面获取
                    // 需要将可见的，放入到 CompilerVisibleProperty 里面
                    // 需要加上 `build_property.` 前缀
                    if (provider.GlobalOptions.TryGetValue("build_property.MyCustomProperty", out var myCustomProperty))
                    {
                        text += " " + myCustomProperty;
                    }

                    var code = @"using System;
namespace LainewihereJerejawwerye
{
    public static class Foo
    {
        public static void F1()
        {
            Console.WriteLine(""" + text + @""");
        }
    }
}";
                    productionContext.AddSource("Demo", code);
                });
        }
```

尝试在 LainewihereJerejawwerye 项目调用一下

```csharp
Foo.F1();
```

然后运行 LainewihereJerejawwerye 项目，可以看到输出了 MyCustomProperty 属性的内容，证明获取 csproj 项目文件里的属性成功

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/d4fee9a801c5d2591162ce4280ac06906029ef48/LainewihereJerejawwerye) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d4fee9a801c5d2591162ce4280ac06906029ef48/LainewihereJerejawwerye) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d4fee9a801c5d2591162ce4280ac06906029ef48
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin d4fee9a801c5d2591162ce4280ac06906029ef48
```

获取代码之后，进入 LainewihereJerejawwerye 文件夹

更多源代码生成，请看官方的 [Source Generators Cookbook](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md)

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
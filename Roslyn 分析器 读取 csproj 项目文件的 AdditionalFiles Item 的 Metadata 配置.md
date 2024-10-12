# Roslyn 分析器 读取 csproj 项目文件的 AdditionalFiles Item 的 Metadata 配置

定义在 ItemGroup 里面的各个引用文件的 Item 可带上自定义的 Metadata 内容，这部分内容需要转换到 AdditionalFiles 的 Metadata 上才能被分析器所获取

<!--more-->
<!-- CreateTime:2024/10/12 07:11:30 -->
<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,编译器,SourceGenerator,分析器,生成代码 -->

在[上一篇](https://blog.lindexi.com/post/IIncrementalGenerator-%E5%A2%9E%E9%87%8F-Source-Generator-%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E5%85%A5%E9%97%A8-%E8%AF%BB%E5%8F%96-csproj-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E7%9A%84%E5%B1%9E%E6%80%A7%E9%85%8D%E7%BD%AE.html )博客告诉大家如何在 IIncrementalGenerator 增量 Source Generator 里读取 csproj 项目文件的属性配置，详细请看：
[IIncrementalGenerator 增量 Source Generator 生成代码入门 读取 csproj 项目文件的属性配置](https://blog.lindexi.com/post/IIncrementalGenerator-%E5%A2%9E%E9%87%8F-Source-Generator-%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E5%85%A5%E9%97%A8-%E8%AF%BB%E5%8F%96-csproj-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E7%9A%84%E5%B1%9E%E6%80%A7%E9%85%8D%E7%BD%AE.html )

在上一篇博客里面，核心是通过配置了 `CompilerVisibleProperty` 让属性可见，如下面代码所示

```xml
  <PropertyGroup>
    <MyCustomProperty>lindexi is doubi</MyCustomProperty>
  </PropertyGroup>

  <ItemGroup>
    <CompilerVisibleProperty Include="MyCustomProperty" />
  </ItemGroup>
```

在分析器里面获取属性内容的核心代码如下

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
                });
```

本文将告诉大家如何获取引用文件的 ItemGroup 里面的 Item 的 Metadata 内容如何获取到。如下面项目文件的代码，定义了名为 PaintStateDiagramMarkdownFile 的 Item 项，此项里面包含了 Link 这个 Metadata 内容

```xml
  <ItemGroup>
    <PaintStateDiagramMarkdownFile Include="..\KereqeewahaihibayNohelqiji\*.txt" Link="Assets\StateDiagrams\%(RecursiveDir)%(Filename)%(Extension)" />
  </ItemGroup>
```

如期望让其 `PaintStateDiagramMarkdownFile` 项在分析器可见，需要将其添加到 AdditionalFiles 里面，如下面代码

```xml
  <ItemGroup>
    <AdditionalFiles Include="@(PaintStateDiagramMarkdownFile)"/>
  </ItemGroup>
```

为了让 Link 这个 Metadata 内容同样被分析器可见，需要在将 PaintStateDiagramMarkdownFile 添加到 AdditionalFiles 时，附带将 Link 带上，更新之后的代码如下

```xml
  <ItemGroup>
    <AdditionalFiles Include="@(PaintStateDiagramMarkdownFile)" Link="%(Link)"/>
  </ItemGroup>
```

再使用 CompilerVisibleItemMetadata 设置 AdditionalFiles 的 Link 也是对分析器可见，代码如下

```xml
  <ItemGroup>
    <CompilerVisibleItemMetadata Include="AdditionalFiles" MetadataName="Link" />
  </ItemGroup>
```

如此即可在后续分析器里面里面使用 AnalyzerConfigOptionsProvider 的 GetOptions 方法获取到 Metadata 信息

以上编辑之后的 csproj 项目文件内容如下

```xml
  <ItemGroup>
    <PaintStateDiagramMarkdownFile Include="..\KereqeewahaihibayNohelqiji\*.txt" Link="Assets\StateDiagrams\%(RecursiveDir)%(Filename)%(Extension)" />

    <AdditionalFiles Include="@(PaintStateDiagramMarkdownFile)" Link="%(Link)"/>

    <CompilerVisibleItemMetadata Include="AdditionalFiles" MetadataName="Link" />
  </ItemGroup>
```

当然了，在真正的 NuGet 打包项目上，常常会将 AdditionalFiles 和 CompilerVisibleItemMetadata 写到 target 或 props 文件里面，而不是放在 csproj 里面。详细请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 获取分析器入门知识

在分析器里面里面，可先收集或用其他方式获取到 AdditionalFiles 内容，将其传入给到 AnalyzerConfigOptionsProvider 的 GetOptions 方法，即可获取到 AnalyzerConfigOptions 对象。再对 AnalyzerConfigOptions 调用 TryGetValue 方法，传入字符串格式是 `build_metadata.AdditionalFiles.[MetadataName]` 即可获取到 Metadata 信息。以上字符串格式的 `[MetadataName]` 还请替换为实际需要获取的值，如本文以上例子里面期望获取到 Link 这个 Metadata 内容，可使用如下代码

```csharp
AnalyzerConfigOptionsProvider analyzerConfigOptionsProvider = ...
AdditionalText additionalText = ...

                        AnalyzerConfigOptions analyzerConfigOptions = analyzerConfigOptionsProvider.GetOptions(additionalText);
                        if (analyzerConfigOptions.TryGetValue("build_metadata.AdditionalFiles.Link", out var link))
                        {
                            
                        }
```

为了更好的说明使用方法，我创建了两个项目，其中一个项目是分析器项目，另一个是控制台项目。我将演示如何获取控制台项目所配置的 PaintStateDiagramMarkdownFile 项的 Link 信息。本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

以下是控制台项目 CujelcijallChearjawjuja 的 csproj 文件的代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PaintStateDiagramMarkdownFile Include="..\KereqeewahaihibayNohelqiji\*.txt" Link="Assets\StateDiagrams\%(RecursiveDir)%(Filename)%(Extension)" />
    <None Include="@(PaintStateDiagramMarkdownFile)"></None>
    <AdditionalFiles Include="@(PaintStateDiagramMarkdownFile)" Link="%(Link)"/>

    <CompilerVisibleItemMetadata Include="AdditionalFiles" MetadataName="Link" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\KereqeewahaihibayNohelqiji\KereqeewahaihibayNohelqiji.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
  </ItemGroup>

</Project>
```

可以看到写了 PaintStateDiagramMarkdownFile 项，且将其添加到 AdditionalFiles 里面，添加的过程中还将 PaintStateDiagramMarkdownFile 的 Link 赋值给到 AdditionalFiles 的 Link 里

最后配置 CompilerVisibleItemMetadata 让 AdditionalFiles 的 Link 信息被分析器可见

中间添加的 `<None Include="@(PaintStateDiagramMarkdownFile)"></None>` 只是一个简单的调试代码，用于让我可以在 VisualStudio 项目里面看到文件而已，和本文实际的演示没有关系

添加分析器 KereqeewahaihibayNohelqiji 项目，分析器项目的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.11.0" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.10.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

以上代码的 EnforceExtendedAnalyzerRules 属性含义请参阅 [Roslyn 分析器 EnforceExtendedAnalyzerRules 属性的作用](https://blog.lindexi.com/post/Roslyn-%E5%88%86%E6%9E%90%E5%99%A8-EnforceExtendedAnalyzerRules-%E5%B1%9E%E6%80%A7%E7%9A%84%E4%BD%9C%E7%94%A8.html )

在 KereqeewahaihibayNohelqiji 项目放入 TextFile.txt 文件用于给 CujelcijallChearjawjuja 项目引用

在 KereqeewahaihibayNohelqiji 编写名为 YelgahainaljinalBehoballdewur 的 IIncrementalGenerator 增量 Source Generator 生成器，代码如下

```csharp
namespace KereqeewahaihibayNohelqiji
{
    [Generator(LanguageNames.CSharp)]
    internal class YelgahainaljinalBehoballdewur : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            // 在这里编写代码
            context.RegisterImplementationSourceOutput(context.AdditionalTextsProvider.Collect().Combine(context.AnalyzerConfigOptionsProvider),
                (productionContext, provider) =>
                {
                    // 这里的代码只有当配置初始化或变更时才会被执行
                    var additionalTextArray = provider.Left;
                    AnalyzerConfigOptionsProvider analyzerConfigOptionsProvider = provider.Right;

                    var stringBuilder = new StringBuilder();
                    for (int i = 0; i < 3; i++)
                    {
                        stringBuilder.Append('"');
                    }
                    stringBuilder.AppendLine();

                    foreach (AdditionalText additionalText in additionalTextArray)
                    {
                        AnalyzerConfigOptions analyzerConfigOptions = analyzerConfigOptionsProvider.GetOptions(additionalText);
                        if (analyzerConfigOptions.TryGetValue("build_metadata.AdditionalFiles.Link", out var link))
                        {
                            stringBuilder.AppendLine($"File={additionalText.Path}");
                            stringBuilder.AppendLine($"Link={link}");
                        }
                    }

                    for (int i = 0; i < 3; i++)
                    {
                        stringBuilder.Append('"');
                    }

                    var code = @"using System;
namespace CujelcijallChearjawjuja
{
    public static class Foo
    {
        public static void F1()
        {
            Console.WriteLine(" + stringBuilder.ToString() + @");
        }
    }
}";
                    productionContext.AddSource("Foo.cs", code);
                });
        }
    }
}
```

如此期望能够将控制台项目里面的 PaintStateDiagramMarkdownFile 作为 Foo 类型的 F1 方法输出控制台信息的内容

编辑控制台项目 CujelcijallChearjawjuja 的 Program.cs 文件，添加如下代码

```csharp
using CujelcijallChearjawjuja;

Foo.F1();
```

运行控制台项目，大概可以看到如下输出内容。如此可以证明此方法可以获取 Item 项里面配置的 Metadata 信息

```
File=C:\lindexi\Workbench\CujelcijallChearjawjuja\..\KereqeewahaihibayNohelqiji\TextFile.txt
Link=Assets\StateDiagrams\TextFile.txt
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/eb119d9e64e387cea847aee79f4509744c349018/Workbench/CujelcijallChearjawjuja) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/eb119d9e64e387cea847aee79f4509744c349018/Workbench/CujelcijallChearjawjuja) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin eb119d9e64e387cea847aee79f4509744c349018
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin eb119d9e64e387cea847aee79f4509744c349018
```

获取代码之后，进入 Workbench/CujelcijallChearjawjuja 文件夹，即可获取到源代码

更多分析器和源代码生成博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
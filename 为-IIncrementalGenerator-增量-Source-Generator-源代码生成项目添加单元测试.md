
# 为 IIncrementalGenerator 增量 Source Generator 源代码生成项目添加单元测试

本文属于 IIncrementalGenerator 增量 Source Generator 源代码生成入门系列博客，本文将和大家介绍如何为源代码生成项目添加单元测试

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

添加单元测试的作用不仅可以用来实现通用的单元测试提高质量的功能，还能用来辅助调试 IIncrementalGenerator 增量 Source Generator 源代码生成项目，从而提高开发效率

传统的类似源代码生成项目的开发调试方式都是需要依赖于另一个项目，通过对另一个项目的构建进行调试测试。通过 Debugger.Break 或 Launch 实现另一个项目构建过程中回到当前 VS 进行调试。详细请参阅之前 walterlv 大佬编写的博客 [使用 Source Generator 在编译你的 .NET 项目时自动生成代码 - walterlv](https://blog.walterlv.com/post/generate-csharp-source-using-roslyn-source-generator)

这样的过程显然对开发效率造成了一定的影响，本文接下来介绍的添加单元测试的方法，将可以实现比较友好的调试。且定制给的调试的内容还可以存放起来作为单元测试的内容，同时单元测试本身的单元功能可以让单元测试项目里面存放不同的多个方向的测试内容，方便调试多个不同的模块

为了方便博客描述，接下来我将创建一个简单的 IIncrementalGenerator 增量 Source Generator 源代码生成项目。我是直接创建名为 YawrofajuGekeyaljilay 控制台项目，然后编辑控制台的 csproj 项目文件，替换为如下代码，进行快速创建的

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.8.0" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

接下来按照官方的例子编写一个特别简单的源代码生成代码，如下面代码

```csharp
using Microsoft.CodeAnalysis;

using System;
using System.Collections.Generic;
using System.Text;

namespace YawrofajuGekeyaljilay
{
    [Generator(LanguageNames.CSharp)]
    public class CodeCollectionIncrementalGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            string source = @"
using System;

namespace YawrofajuGekeyaljilay
{
    public static partial class Program
    {
        public static void HelloFrom(string name)
        {
            Console.WriteLine($""Says: Hi from '{name}'"");
        }
    }
}
";

            context.RegisterPostInitializationOutput(initializationContext =>
            {
                initializationContext.AddSource("GeneratedSourceTest", source);
            });
        }
    }
}
```

基础逻辑准备完成之后，接下来即可为此源代码生成项目创建单元测试项目

为了方便和效率起见，我依然是通过创建控制台项目编辑 csproj 项目文件替换为如下代码的方式快速创建单元测试项目

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
    <PackageReference Include="MSTest.TestAdapter" Version="3.2.0" />
    <PackageReference Include="MSTest.TestFramework" Version="3.2.0" />

    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.8.0" />
    <PackageReference Include="Microsoft.CodeAnalysis.Common" Version="4.8.0" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Workspaces" Version="4.8.0" />

    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Analyzer.Testing.MSTest" Version="1.1.1" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.CodeFix.Testing.MSTest" Version="1.1.1" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.CodeRefactoring.Testing.MSTest" Version="1.1.1" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.SourceGenerators.Testing.MSTest" Version="1.1.1" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\YawrofajuGekeyaljilay\YawrofajuGekeyaljilay.csproj" />
  </ItemGroup>

</Project>
```

以上的单元测试项目和传统的单元测试项目不同的在于添加了以下这些额外的引用库

```xml
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.8.0" />
    <PackageReference Include="Microsoft.CodeAnalysis.Common" Version="4.8.0" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Workspaces" Version="4.8.0" />

    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Analyzer.Testing.MSTest" Version="1.1.1" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.CodeFix.Testing.MSTest" Version="1.1.1" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.CodeRefactoring.Testing.MSTest" Version="1.1.1" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.SourceGenerators.Testing.MSTest" Version="1.1.1" />
```

完成基础的项目构建之后，接下来可以对源代码生成编写单元测试。以下例子将创建名为 GeneratorTests 的单元测试用来演示如何对源代码生成进行测试或调试

新建 GeneratorTests 类型，先添加辅助的方法，代码如下

```csharp
    private static CSharpCompilation CreateCompilation(string source)
        => CSharpCompilation.Create("compilation",
            new[] { CSharpSyntaxTree.ParseText(source) },
            new[] { MetadataReference.CreateFromFile(typeof(Binder).GetTypeInfo().Assembly.Location) },
            new CSharpCompilationOptions(OutputKind.ConsoleApplication));
```

以上的辅助方法的作用就是可以让单元测试在传入一段代码时，转换为 CSharpCompilation 类型。同时添加上默认的 System.Runtime 的引用，防止一些基础类型找不到

完成以上辅助方法之后，可以编写 SimpleGeneratorTest 单元测试方法，开始的代码如下，先传入一段代码用来作为测试的输入

```csharp
[TestClass]
public class GeneratorTests
{
    [TestMethod]
    public void SimpleGeneratorTest()
    {
        Compilation inputCompilation = CreateCompilation(@"
namespace YawrofajuGekeyaljilay
{
    public static class Program
    {
        public static void Main(string[] args)
        {
        }
    }
}
");
        // 忽略其他代码
    }
}
```

通过以上代码就可以在单元测试里面定义多个不同的输入代码源，从而使用不同的代码输入源进行测试或调试源代码生成项目

接下来创建用来测试的 CodeCollectionIncrementalGenerator 类型

```csharp
        var codeCollectionIncrementalGenerator = new CodeCollectionIncrementalGenerator();
```

再创建用来辅助测试的 CSharpGeneratorDriver 类型

```csharp
        var driver = CSharpGeneratorDriver.Create(codeCollectionIncrementalGenerator);
```

在 CSharpGeneratorDriver 的 Create 方法里面，是允许传入多个 IIncrementalGenerator 的，这就意味着你可以同时对多个 IIncrementalGenerator 实例进行测试

完成创建之后，接下来就是开始执行，代码如下

```csharp
        driver.RunGeneratorsAndUpdateCompilation(inputCompilation, out var outputCompilation, out var diagnostics);
```

此 RunGeneratorsAndUpdateCompilation 方法将会通过方法返回执行完成之后，现在所有的 Compilation 和过程产生的 Diagnostic 集合。以上代码的 `outputCompilation` 的 SyntaxTrees 不仅包含原本输入的 Compilation 里的代码也包含源代码生成器添加的源代码

拿到运行结果之后，即可继续编写代码测试结果，如下面代码

```csharp
        Assert.AreEqual(true, outputCompilation.ContainsSymbolsWithName("HelloFrom"));
```

也可以使用下面代码展开所有的代码，通过字符串比对之类的，判断生成是否正确，或者进行调试，了解生成的内容

```csharp
        foreach (var outputCompilationSyntaxTree in outputCompilation.SyntaxTrees)
        {
            var text = outputCompilationSyntaxTree.GetText();
        }
```

如果只是想要获取生成的代码，可以取 RunGeneratorsAndUpdateCompilation 方法的返回值，此方法的返回值也是一个 GeneratorDriver 对象。返回自身类型在这里不是为了方便做链调用，而是使用不可变思想，即任何的更改都会创建出新的对象，不会对原有的对象进行更改。不可变思想在 Roslyn 里贯穿实现，从而造就了 Roslyn 如此复杂却又方便进行调试。取到返回的 GeneratorDriver 的 GetRunResult 即可获取到 GeneratorDriverRunResult 类型对象，通过 GeneratorDriverRunResult 的 GeneratedTrees 即可获取到只有源代码生成项目生成的代码

```csharp
        GeneratorDriver driver = CSharpGeneratorDriver.Create(codeCollectionIncrementalGenerator);
        driver = driver.RunGeneratorsAndUpdateCompilation(inputCompilation, out var outputCompilation, out var diagnostics);

        var generatorDriverRunResult = driver.GetRunResult();
        Assert.AreEqual(1, generatorDriverRunResult.GeneratedTrees.Length);
```

在一些比较复杂的项目上，可能需要参与测试的代码会需要使用到各种各样的 dotnet 引用，此时适合将整个 dotnet 运行时都添加进入引用，防止找不到引用导致失败。以下是我添加的辅助类型，用来将整个 dotnet 的基础库添加到引用

```csharp
internal static class MetadataReferenceProvider
{
    public static IReadOnlyList<MetadataReference> GetDotNetMetadataReferenceList()
    {
        if (_cacheList is not null)
        {
            return _cacheList;
        }

        var metadataReferenceList = new List<MetadataReference>();
        var assembly = Assembly.Load("System.Runtime");
        foreach (var file in Directory.GetFiles(Path.GetDirectoryName(assembly.Location)!, "*.dll"))
        {
            try
            {
                metadataReferenceList.Add(MetadataReference.CreateFromFile(file));
            }
            catch
            {
                // 忽略
            }
        }

        _cacheList = metadataReferenceList;
        return _cacheList;
    }

    private static IReadOnlyList<MetadataReference>? _cacheList;
}
```

使用例子如下

```csharp
    private static CSharpCompilation CreateCompilation(string source)
    {
        return CSharpCompilation.Create("compilation",
            new[] { CSharpSyntaxTree.ParseText(source) },
            new[]
            {
            	// 添加业务方的程序集
                MetadataReference.CreateFromFile(typeof(Foo).Assembly.Location), 
            }
            // 加上整个 dotnet 的基础库
            .Concat(MetadataReferenceProvider.GetDotNetMetadataReferenceList()),
            new CSharpCompilationOptions(OutputKind.ConsoleApplication));
    }
```

额外的，大家也看到本身的例子里面的输入是靠代码里面编写字符串进行实现的。这样的方法会导致编写代码字符串的难度，且写错了可能自己还不知道，从而导致了单元测试反而影响调试效率。每次都在外面写完拷贝字符串进来，看起来实现也不友好。解决方法就是添加正常的代码给到自己的项目里面，然后直接将代码文件的内容读取出来。比如说将代码文件输出到输出文件夹，或者是将代码文件嵌入到程序集，走程序集读取资源的方式。下面的例子是我创建一个名为 TestCode.cs 的文件，我在 csproj 里面额外将此文件设置作为嵌入的资源，如下面代码

```xml
  <ItemGroup>
    <EmbeddedResource Include="TestCode.cs" />
  </ItemGroup>
```

于是代码里面就可以读取程序集嵌入资源，从而读取到代码文件里面的内容作为字符串进行输入

```csharp
internal static class TestCodeProvider
{
    public static string GetTestCode()
    {
        var manifestResourceStream = typeof(TestCodeProvider).Assembly.GetManifestResourceStream("程序集名.TestCode.cs")!;
        var streamReader = new StreamReader(manifestResourceStream);
        return streamReader.ReadToEnd();
    }
}
```

另外的常见问题就是默认开启了 ImplicitUsings 导致 System 之类的命名空间没有引用，进而在单元测试里面，导致源代码生成项目解析失败。在正式使用的时候，需要先确保所有的引用加载上，且作为输入源的代码都能正常构建通过

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/3b7623ad46e80e8cc88a51e8084339ac29937b64/YawrofajuGekeyaljilay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/3b7623ad46e80e8cc88a51e8084339ac29937b64/YawrofajuGekeyaljilay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 3b7623ad46e80e8cc88a51e8084339ac29937b64
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 3b7623ad46e80e8cc88a51e8084339ac29937b64
```

获取代码之后，进入 YawrofajuGekeyaljilay 文件夹

更多关于源代码生成博客请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
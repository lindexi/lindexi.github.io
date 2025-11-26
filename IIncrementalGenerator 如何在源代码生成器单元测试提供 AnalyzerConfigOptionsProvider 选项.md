# IIncrementalGenerator 如何在源代码生成器单元测试提供 AnalyzerConfigOptionsProvider 选项

本文属于 IIncrementalGenerator 增量 Source Generator 源代码生成入门系列博客，本文将和大家介绍如何为源代码生成项目添加的单元测试里面提供 AnalyzerConfigOptionsProvider 选项

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文是 [为 IIncrementalGenerator 增量 Source Generator 源代码生成项目添加单元测试](https://blog.lindexi.com/post/%E4%B8%BA-IIncrementalGenerator-%E5%A2%9E%E9%87%8F-Source-Generator-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95.html ) 的后续。在上文介绍了如何给增量 Source Generator 源代码生成项目添加单元测试，本文将在此基础上，告诉大家如何提供 AnalyzerConfigOptionsProvider 选项

先来看看一个简单的源代码生成器的例子，以下的代码将根据配置的 FooProperty 属性决定生成的代码内容

```csharp
[Generator(LanguageNames.CSharp)]
public class IncrementalGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        IncrementalValueProvider<string> configurationProvider = context.AnalyzerConfigOptionsProvider.Select((t, _) =>
        {
            var globalOptions = t.GlobalOptions;
            if (globalOptions.TryGetValue("build_property.FooProperty", out var property))
            {
                return property;
            }

            return null;
        });

        context.RegisterSourceOutput(configurationProvider, (productionContext, configurationProperty) =>
        {
            productionContext.AddSource("GeneratedCode.cs",
                $$"""
                  using System;
                  
                  namespace LurlelnarkallChijurjeaqelba
                  {
                      public static class GeneratedCode
                      {
                          public static void Print()
                          {
                              Console.WriteLine("配置的属性 ｛｛configurationProperty｝｝");
                          }
                      }
                  }
                  """);
        });
    }
}
```

注： 为了让我的博客引擎开森，以上代码部分花括号被我替换为了全角花括号。大家在使用的时候需要将全角花括号替换为半角花括号

以上的 `build_property.FooProperty` 就是获取某个属性配置的写法，正常来说是需要依靠 `CompilerVisibleProperty` 指定具体的属性名才能被源代码生成器访问到的，详细请参阅 [IIncrementalGenerator 增量 Source Generator 生成代码入门 读取 csproj 项目文件的属性配置](https://blog.lindexi.com/post/IIncrementalGenerator-%E5%A2%9E%E9%87%8F-Source-Generator-%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E5%85%A5%E9%97%A8-%E8%AF%BB%E5%8F%96-csproj-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E7%9A%84%E5%B1%9E%E6%80%A7%E9%85%8D%E7%BD%AE.html )

在正式项目里面大概的写法如下：

```xml
  <PropertyGroup>
    <FooProperty>lindexi is doubi</FooProperty>
  </PropertyGroup>

  <ItemGroup>
    <CompilerVisibleProperty Include="FooProperty" />
  </ItemGroup>
```

以上代码既可以写到 csproj 项目文件里面，也可以被放在带在 NuGet 包里的 props 文件里面。如果对此机制感觉到陌生，还请参阅 [dotnet 源代码生成器分析器入门](https://blog.lindexi.com/post/dotnet-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8%E5%88%86%E6%9E%90%E5%99%A8%E5%85%A5%E9%97%A8.html )
<!-- [dotnet 源代码生成器分析器入门 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18786647 ) -->

单元测试里面，需要使用 `CSharpGeneratorDriver.Create` 的重载方法传入 AnalyzerConfigOptionsProvider 类型参数

由于 AnalyzerConfigOptionsProvider 是抽象的，我添加了如下代码用于辅助测试

```csharp
internal class TestAnalyzerConfigOptionsProvider : AnalyzerConfigOptionsProvider
{
    public TestAnalyzerConfigOptionsProvider(Dictionary<string, string> configOptions)
    {
        var testAnalyzerConfigOptions = new TestAnalyzerConfigOptions(configOptions);
        GlobalOptions = testAnalyzerConfigOptions;
    }

    public override AnalyzerConfigOptions GetOptions(SyntaxTree tree)
    {
        return GlobalOptions;
    }

    public override AnalyzerConfigOptions GetOptions(AdditionalText textFile)
    {
        return GlobalOptions;
    }

    public override AnalyzerConfigOptions GlobalOptions { get; }
}

internal class TestAnalyzerConfigOptions : AnalyzerConfigOptions
{
    public TestAnalyzerConfigOptions(Dictionary<string, string> configOptions)
    {
        _configOptions = configOptions;
    }

    private readonly Dictionary<string, string> _configOptions;

    public override bool TryGetValue(string key, [NotNullWhen(true)] out string? value)
    {
        return _configOptions.TryGetValue(key, out value);
    }
}
```

通过 TestAnalyzerConfigOptionsProvider 辅助代码，可以使用字典表示将要注入到测试里面的属性，代码如下

```csharp
        // 创建 AnalyzerConfigOptions
        var configOptions = new Dictionary<string, string>
        {
            ["build_property.FooProperty"] = "Test",
        };
        var analyzerConfigOptionsProvider = new TestAnalyzerConfigOptionsProvider(configOptions);
```

将 `configOptions` 传入到 `CSharpGeneratorDriver.Create` 方法里面，代码如下

```csharp
        var compilation = CreateCompilation(...);
        var generator = new IncrementalGenerator();

        GeneratorDriver driver = CSharpGeneratorDriver.Create([generator.AsSourceGenerator()], optionsProvider: analyzerConfigOptionsProvider);
```

这里需要让 IncrementalGenerator 通过 AsSourceGenerator 扩展方法转换为 ISourceGenerator 类型，才能传入此重载方法里面

完成以上步骤之后，就可以调用 `GeneratorDriver.RunGenerators` 开始执行源代码生成器

```csharp
        driver = driver.RunGenerators(compilation);
```

以下是示例的单元测试

```csharp

[TestClass]
public class IncrementalGeneratorTest
{
    [TestMethod]
    public void Test()
    {
        var testCode =
            """
            using System;
            
            namespace LurlelnarkallChijurjeaqelba;
            """;

        var compilation = CreateCompilation(testCode);
        var generator = new IncrementalGenerator();

        // 创建 AnalyzerConfigOptions
        var configOptions = new Dictionary<string, string>
        {
            ["build_property.FooProperty"] = "Test",
        };
        var analyzerConfigOptionsProvider = new TestAnalyzerConfigOptionsProvider(configOptions);

        GeneratorDriver driver = CSharpGeneratorDriver.Create([generator.AsSourceGenerator()], optionsProvider: analyzerConfigOptionsProvider);
        driver = driver.RunGenerators(compilation);

        var runResult = driver.GetRunResult();
        Assert.HasCount(1, runResult.GeneratedTrees);
        foreach (var generatedTree in runResult.GeneratedTrees)
        {
            var generatedCode = generatedTree.ToString();
            Debug.WriteLine(generatedCode);

            if (generatedTree.FilePath.EndsWith("GeneratedCode.cs"))
            {
                var expected =
                    """
                    using System;
                    
                    namespace LurlelnarkallChijurjeaqelba
                    {
                        public static class GeneratedCode
                        {
                            public static void Print()
                            {
                                Console.WriteLine("配置的属性 Test");
                            }
                        }
                    }
                    """;

                // 防止拉取 git 时出现的 \r\n 不匹配问题。能够解决一些拉取 git 的奇怪的坑，也就是在我电脑上跑的好好的，但为什么在你电脑上就炸了
                expected = expected.Replace("\r\n", "\n");
                Assert.AreEqual(expected, generatedCode.Replace("\r\n", "\n"));
            }
        }
    }

    private static CSharpCompilation CreateCompilation(string source)
        => CSharpCompilation.Create("compilation",
            new[] { CSharpSyntaxTree.ParseText(source, path: "Foo.cs") },
            new MetadataReference[]
            {
                // 如果缺少引用，那将会导致单元测试有些符号无法寻找正确，从而导致解析失败
                // 在这里添加你自己的依赖库的引用
            }
            // 加上整个 dotnet 的基础库
            .Concat(MetadataReferenceProvider.GetDotNetMetadataReferenceList()),
            new CSharpCompilationOptions(OutputKind.ConsoleApplication));
}

internal class TestAnalyzerConfigOptionsProvider : AnalyzerConfigOptionsProvider
{
    public TestAnalyzerConfigOptionsProvider(Dictionary<string, string> configOptions)
    {
        var testAnalyzerConfigOptions = new TestAnalyzerConfigOptions(configOptions);
        GlobalOptions = testAnalyzerConfigOptions;
    }

    public override AnalyzerConfigOptions GetOptions(SyntaxTree tree)
    {
        return GlobalOptions;
    }

    public override AnalyzerConfigOptions GetOptions(AdditionalText textFile)
    {
        return GlobalOptions;
    }

    public override AnalyzerConfigOptions GlobalOptions { get; }
}

internal class TestAnalyzerConfigOptions : AnalyzerConfigOptions
{
    public TestAnalyzerConfigOptions(Dictionary<string, string> configOptions)
    {
        _configOptions = configOptions;
    }

    private readonly Dictionary<string, string> _configOptions;

    public override bool TryGetValue(string key, [NotNullWhen(true)] out string? value)
    {
        return _configOptions.TryGetValue(key, out value);
    }
}

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

本文的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/28cf0fb972be64983e4e5eb91d9dd62930f2d49d/Roslyn/LurlelnarkallChijurjeaqelba) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/28cf0fb972be64983e4e5eb91d9dd62930f2d49d/Roslyn/LurlelnarkallChijurjeaqelba) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 28cf0fb972be64983e4e5eb91d9dd62930f2d49d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 28cf0fb972be64983e4e5eb91d9dd62930f2d49d
```

获取代码之后，进入 Roslyn/LurlelnarkallChijurjeaqelba 文件夹，即可获取到源代码
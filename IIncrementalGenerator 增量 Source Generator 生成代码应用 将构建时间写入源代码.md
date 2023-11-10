# IIncrementalGenerator 增量 Source Generator 生成代码应用 将构建时间写入源代码

本文将和大家介绍一个 IIncrementalGenerator 增量 Source Generator 生成代码技术的应用例子，将当前的构建时间写入到代码里面。这个功能可以比较方便实现某些功能的开关，比如说设置某个功能自动在具体应用发布之后过一段时间就失效等功能

<!--more-->
<!-- CreateTime:2023/11/8 20:40:08 -->

<!-- 发布 -->
<!-- 博客 -->

在不使用 IIncrementalGenerator 增量 Source Generator 生成代码技术时，也可以方便的实现在代码里面了解应用的构建时间，请看 [Roslyn MSBuild 在构建完成之后 将构建时间写入到输出文件](https://blog.lindexi.com/post/Roslyn-MSBuild-%E5%9C%A8%E6%9E%84%E5%BB%BA%E5%AE%8C%E6%88%90%E4%B9%8B%E5%90%8E-%E5%B0%86%E6%9E%84%E5%BB%BA%E6%97%B6%E9%97%B4%E5%86%99%E5%85%A5%E5%88%B0%E8%BE%93%E5%87%BA%E6%96%87%E4%BB%B6.html )

本文将介绍的方法可以完全通过预编译方式，将构建时间写固定到代码里面，参与构建。对比以上博客最大的不同在于不需要引入额外的本地文件

本文使用的代码大量从 [C# Source Generators: How to get build information?](https://steven-giesel.com/blogPost/cec8df6e-b271-4b4c-8ff6-e9f3aa5e26a1 ) 博客里面拷贝，感谢 [Steven Giesel](https://github.com/linkdotnet) 大佬提供的方法

根据 [IIncrementalGenerator 增量 Source Generator 生成代码技术](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html) 的入门博客，咱将创建两个项目，一个用来测试源代码生成效果，一个用来制作核心的源代码生成。本文将跳过入门级的项目初始化介绍，如对分析器项目的创建有疑惑，还请参阅[入门博客](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html)了解更多内容

我将在本文末尾放入所使用的代码的下载方式，预计代码拉下来是可以非常方便运行和调试

在源代码生成项目里面，也就是分析器项目里面新建一个用来辅助放入构建信息的类型，比如本文使用 FooGenerator 作为生成器类型的命名，定义如下

```csharp
    [Generator(LanguageNames.CSharp)]
    public class FooGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
        }
    }
```

先订阅项目的配置，根据项目的 Options 配置即可生成包含构建信息的代码

```csharp
    [Generator(LanguageNames.CSharp)]
    public class FooGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {
            var compilerOptions = context.CompilationProvider.Select((s, _) => s.Options);
        }
    }
```

完成订阅之后即可设置源代码生成的输出，如以下代码，生成了一个名为 BuildInformation 的静态类，且此静态类还没有包含在任何的命名空间里面

```csharp
            context.RegisterSourceOutput(compilerOptions, static (productionContext, options) =>
            {
                var code = $@"
using System;
using System.Globalization;

public static class BuildInformation
｛｛
    /// <summary>
    /// Returns the build date (UTC).
    /// </summary>
    public static readonly DateTime BuildAt = DateTime.ParseExact(""{DateTime.UtcNow:O}"", ""O"", CultureInfo.InvariantCulture, DateTimeStyles.RoundtripKind);
    /// <summary>
    /// Returns the platform.
    /// </summary>
    public const string Platform = ""{options.Platform}"";
    /// <summary>
    /// Returns the configuration.
    /// </summary>
    public const string Configuration = ""{options.OptimizationLevel}"";
｝｝
";

                productionContext.AddSource("LinkDotNet.BuildInformation.g", code);
            });
```

如此即可生成一个包含构建时间以及构建时使用的平台，以及构建配置是 Debug 还是 Release 的源代码

使用源代码生成器生成的代码的项目即可直接使用 BuildInformation 类型获取到对应的构建信息，如以下代码例子

```csharp
Console.WriteLine($"BuildAt={BuildInformation.BuildAt}");
Console.WriteLine($"Platform={BuildInformation.Platform}");
Console.WriteLine($"Configuration={BuildInformation.Configuration}");
```

运行的输出内容大概如下

```csharp
BuildAt=2023/11/9 13:41:29
Platform=AnyCpu
Configuration=Release
```

如此即可很方便通过源代码生成技术将构建信息写入到代码里面，让业务方根据写入的构建信息决定具体的行为

如果大家对这个将构建写入到代码的功能特别感兴趣，但自己不想再去创建分析器，可以使用大佬编写好的 NuGet 库。使用非常方便，只需要安装 [LinkDotNet.BuildInformation](https://www.nuget.org/packages/LinkDotNet.BuildInformation) 即可，安装完成之后即可在自己的项目里面使用 BuildInformation 类型获取构建信息

安装 [LinkDotNet.BuildInformation](https://www.nuget.org/packages/LinkDotNet.BuildInformation) 库既可以通过 NuGet 管理器安装，也可以编辑 csproj 项目文件添加以下代码进行安装

```xml
<PackageReference Include="LinkDotNet.BuildInformation" Version="0.4.2" />
```

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/2e02c5f1bdfeb3e53578e6ff63aa560f3ce5a51a/QewelechairWhegalnarlaywhe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2e02c5f1bdfeb3e53578e6ff63aa560f3ce5a51a/QewelechairWhegalnarlaywhe) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2e02c5f1bdfeb3e53578e6ff63aa560f3ce5a51a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2e02c5f1bdfeb3e53578e6ff63aa560f3ce5a51a
```

获取代码之后，进入 QewelechairWhegalnarlaywhe 文件夹
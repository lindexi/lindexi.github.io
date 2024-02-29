默认的源代码生成器所生成的代码都是没有直接存放到项目文件夹里面的，不受源代码管理工具管理，对使用方的开发者来说很难直接阅读或查找到 Source Generator 生成的源代码。本文将和大家介绍如何使用 EmitCompilerGeneratedFiles 属性配置将生成的代码保存到本地文件

<!--more-->


<!-- CreateTime:2024/2/7 9:56:37 -->
<!-- 发布 -->
<!-- 博客 -->

将 Source Generator 生成的源代码保存到本地，只需设置 EmitCompilerGeneratedFiles 属性，在 csproj 项目文件里面进行设置，代码如下

```xml
  <PropertyGroup>
    <EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>
  </PropertyGroup>
```

完成此配置之后，将会自动将源代码生成器所生成的代码存放到本地文件夹里面。默认的生成源代码将会存放到 `$(IntermediateOutputPath)\generated` 文件夹里面，这里的 `$(IntermediateOutputPath)` 由 `obj\$(Configuration)\$(TargetFramework.ToLowerInvariant())\` 构成，调试下的输出大概是 `obj\Debug\net8.0\` 等类似的文件夹里

接着将会拼接上源代码生成器分析器项目的程序集名与具体的源代码生成类型，最后加上源代码生成器 AddSource 时设置的 `hintName` 作为文件名。如对于本文例子里面，在名为 YaijowhelawFerhecarnal 的分析器项目里面的名为 `YaijowhelawFerhecarnal.CodeCollectionIncrementalGenerator` 的生成器类型使用如下代码所生成的名为 GeneratedSourceTest 的代码，将会存放到 `obj\Debug\net8.0\generated\YaijowhelawFerhecarnal\YaijowhelawFerhecarnal.CodeCollectionIncrementalGenerator\GeneratedSourceTest.cs` 文件夹里面

```csharp
namespace YaijowhelawFerhecarnal
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

如果期望自己指定保存的文件夹，可以自行设置 EmitCompilerGeneratedFiles 属性，如以下代码

```xml
  <PropertyGroup>
    <CompilerGeneratedFilesOutputPath>Generated\$(TargetFramework)</CompilerGeneratedFilesOutputPath>
  </PropertyGroup>
```

以上代码之所以拼接上 TargetFramework 是因为期望默认处理多框架的文件冲突问题，源代码生成器会在多框架下分别执行，为每个框架生成独立的代码。如果在多框架项目下没有配置加上 TargetFramework 将会造成生成的源代码存放的文件冲突

上面代码添加之后，预计将会导致构建不通过，一般的保存信息如下

```
error CS0111: 类型“Program”已定义了一个名为“HelloFrom”的具有相同参数类型的成员
```

这是因为设置放在 `Generated\$(TargetFramework)` 会被 csproj 默认作为源代码引用，导致原本源代码生成器生成的代码已经在内存里面被引用一次，现在源代码生成器输出的文件又被再次引用，导致了最终构建不通过

解决方法就是去掉对 CompilerGeneratedFilesOutputPath 的文件的引用，确保只有引用源代码生成器在内存的一份代码，如以下代码

```xml
  <ItemGroup>
    <!-- 添加了内存里面的文件，不应该添加磁盘的，否则添加两份 -->
    <Compile Remove="$(CompilerGeneratedFilesOutputPath)/**/*.cs" />
  </ItemGroup>
```

通过以上的方式即可让源代码生成器所生成的文件输出到本地文件里面，方便将生成的代码签进源代码版本控制里面，如 git 等里面，也方便进行静态代码阅读和代码审查

更多关于源代码生成博客请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/522923c0de0d436f6f5a44dffd0b99a325bc5a64/YaijowhelawFerhecarnal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/522923c0de0d436f6f5a44dffd0b99a325bc5a64/YaijowhelawFerhecarnal) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 522923c0de0d436f6f5a44dffd0b99a325bc5a64
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 522923c0de0d436f6f5a44dffd0b99a325bc5a64
```

获取代码之后，进入 YaijowhelawFerhecarnal 文件夹

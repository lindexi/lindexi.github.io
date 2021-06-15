
# Roslyn 如何获得一个类的引用

本文告诉大家如何在 Rosyln 编译一个文件，获得这个文件的类的命名空间

<!--more-->


<!-- CreateTime:2018/7/18 11:31:41 -->

<!-- csdn -->
<!-- 标签：Roslyn,MSBuild,编译器 -->
<!-- 发布 -->

在 C# 代码里面，大部分的代码都是在开始定义了 using 引用命名空间，本文将告诉大家如何使用 Roslyn 分析获取类文件里面引用的命名空间

在开始之前，先使用 NuGet 安装必要的库，如 Microsoft.CodeAnalysis.Compilers 库。在使用 SDK 风格的项目格式，可以编辑 csproj 文件，添加如下代码

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Compilers" Version="3.10.0" />
  </ItemGroup>
```

上面代码使用了 3.10 版本，这个版本支持了 C# 9 的语法

本文使用的是 .NET 5 版本，项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.Compilers" Version="3.10.0" />
  </ItemGroup>

</Project>
```

本文将在 Program.cs 文件里面编写获取的逻辑，分析的文件就是 Program.cs 文件。在 Main 函数里面，使用如下代码获取 Program.cs 的代码

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var file = @"..\..\..\Program.cs";

            file = Path.GetFullPath(file);
            var text = File.ReadAllText(file);
        }
    }
```

以上代码拿到的 text 就是 Program.cs 的代码

通过 `Microsoft.CodeAnalysis.CSharp.CSharpSyntaxTree` 静态类的 ParseText 可以读取到某个传入字符串的语法树，也就是调用此方法相当于进行了 Rolsyn 的分析。在读取出来了语法树，还需要编写分析的代码，分析代码的方法就是编写一个继承 CSharpSyntaxWalker 的类用来作为分析的辅助类

按照约定，咱编写 ModelCollector 类，代码如下

```csharp
    class ModelCollector : CSharpSyntaxWalker
    {

    }
```

继承 CSharpSyntaxWalker 的优势在于有很多代码都可以使用预定义的代码，而咱只需要按照访问者模式编写咱的业务逻辑代码就可以

使用 ModelCollector 类访问语法树代码如下

```csharp
            var modelCollector = new ModelCollector();
            modelCollector.Visit(tree.GetRoot());
```

在 Roslyn 开发，对 C# 语言编写的文件，将会被读为语法树，但是语法树很复杂，此时就需要一个辅助的类去读取对应的逻辑。使用继承 CSharpSyntaxWalker 的类作为辅助类，通过访问者模式的方法去读取，就是 Roslyn 开发推荐的方法

调用 Visit 方法就可以开始进行语法分析，或者进行语法修改添加代码等

但是 ModelCollector 类还没有任何的代码，期望获取当前类文件的 using 引用文件，可以通过在 ModelCollector 重写 VisitUsingDirective 方法的方式获取

```csharp
    class ModelCollector : CSharpSyntaxWalker
    {
        public override void VisitUsingDirective(UsingDirectiveSyntax node)
        {
            Debug.WriteLine(node.Name.ToFullString());
            base.VisitUsingDirective(node);
        }
    }
```

以上的 VisitUsingDirective 方法将会在每一次的 `using` 进入时被调用，也只有是作为命名空间引用的 using 才会进入

如 Program.cs 的代码如下

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
```

那么 VisitUsingDirective 将会进来 6 次，分别是以上的各个 using 语句

详细还请拉下代码，在 VisitUsingDirective 方法添加断点

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c41bfd2d9cc6a82fdc806f8e82f8e929026077b9/KeneenejajiqairCalllebolayere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c41bfd2d9cc6a82fdc806f8e82f8e929026077b9/KeneenejajiqairCalllebolayere) 欢迎小伙伴访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c41bfd2d9cc6a82fdc806f8e82f8e929026077b9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 KeneenejajiqairCalllebolayere 文件夹

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
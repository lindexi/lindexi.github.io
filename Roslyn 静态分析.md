# Roslyn 静态分析

本文告诉大家如何使用 Roslyn 分析代码。

<!--more-->
<!-- CreateTime:2018/8/29 9:10:19 -->


<!-- 标签：Roslyn,MSBuild,编译器 -->

首先创建一个项目，项目使用.net Framework 4.6.2 ，控制台项目。然后需要安装一些需要的库

## Nuget 安装

打开 Nuget 安装下面两个库

 - Microsoft.CodeAnalysis.CSharp

 - Microsoft.CodeAnalysis.CSharp.Workspaces

 - Newtonsoft.Json

## 使用

下面来写简单的代码

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace TrrluujHlcdyqa
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("hellow");
        }
    }

    class Foo
    {
        public string KiqHns { get; set; }
    }
}
```

对上面的代码分析

首先需要把上面的代码放在字符串

然后创建分析代码，读取代码。

```csharp
   class ModelCollector : CSharpSyntaxWalker
    {
        public readonly Dictionary<string, List<string>> Models = new Dictionary<string, List<string>>();
        public override void VisitPropertyDeclaration(PropertyDeclarationSyntax node)
        {
            var classnode = node.Parent as ClassDeclarationSyntax;
            if (classnode != null && !Models.ContainsKey(classnode.Identifier.ValueText))
            {
                Models.Add(classnode.Identifier.ValueText, new List<string>());
            }

            Models[classnode.Identifier.ValueText].Add(node.Identifier.ValueText);
        }
    }

```

```csharp
   class Program
    {
        static void Main(string[] args)
        {
            string str = @"
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace TrrluujHlcdyqa
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine(""hellow"");
        }
    }

    class Foo
    {
        public string KiqHns { get; set; }
    }
}";

            var tree = CSharpSyntaxTree.ParseText(str);

            var root = (CompilationUnitSyntax)tree.GetRoot();
            var modelCollector = new ModelCollector();
            modelCollector.Visit(root);
            Console.WriteLine(JsonConvert.SerializeObject(modelCollector.Models));

        }
    }
```

这时输出`{"Foo":["KiqHns"]}`

上面的代码从 https://stackoverflow.com/a/22881532/6116637 学的

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

参见：

[通过Roslyn构建自己的C#脚本（更新版） - 天方 - 博客园](http://www.cnblogs.com/TianFang/p/6939723.html )

[专栏：Roslyn 入门 - CSDN博客](https://blog.csdn.net/column/details/23159.html )

[Getting Started with Roslyn Analyzers ](https://docs.microsoft.com/en-us/visualstudio/extensibility/getting-started-with-roslyn-analyzers )

[代码分析 - 借助与 NuGet 集成的 Roslyn 代码分析来生成和部署库](https://msdn.microsoft.com/zh-cn/magazine/mt573715.aspx )

[roslyn-analyzers/ReadMe.md at master · dotnet/roslyn-analyzers](https://github.com/dotnet/roslyn-analyzers/blob/master/src/MetaCompilation.Analyzers/Core/ReadMe.md )

[In-memory C# compilation and .dll generation using Roslyn ](https://josephwoodward.co.uk/2016/12/in-memory-c-sharp-compilation-using-roslyn )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

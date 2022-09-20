
# dotnet 使用 IndentedTextWriter 辅助生成代码时生成带缩进的内容

随着源代码生成的越来越多的应用，自然也遇到了越来越多开发上的坑，例如源代码的缩进是一个绕不过去的问题。如果源代码生成是人类可见的代码，我期望生成的代码最好是比较符合人类编写代码的规范。为了能让人类在阅读机器生成的代码的时候，不会想着拿刀砍那个编写代码生成代码的开发者，最好，或者说至少代码也应该有个缩进和换行吧。本文将安利大家通过 IndentedTextWriter 这个辅助类，用来辅助生成带缩进的内容

<!--more-->


<!-- CreateTime:2022/9/19 8:23:29 -->


<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：Roslyn,MSBuild,代码生成 -->

使用 IndentedTextWriter 辅助类核心的用途在于自动加上缩进，缩进的等级由代码设置，可以通过加等和减等控制缩进等级。缩进等级是文本排版的概念，约等于缩进多少个空格

在开始使用 IndentedTextWriter 之前，需要先引用命名空间

```csharp
using System.CodeDom.Compiler;
using System.Text;
```

初始化 IndentedTextWriter 需要传入一个 TextWriter 对象，用来当成写入输出的内容。有趣的是 IndentedTextWriter 类型自己也是继承 TextWriter 的类型，也就是可以用 IndentedTextWriter 类型进行无限套娃

大家都知道，继承 TextWriter 的类型用的多的有 StringWriter 和 StreamWriter 这两个。其中 StringWriter 用于辅助写入 StringBuilder 对象，基本输出限制在内存里面。而 StreamWriter 则是输出到 Stream 里，至于这个 Stream 是文件还是网络，那就看你的心情了

作为例子，咱使用 StringWriter 作为 IndentedTextWriter 的输出。对 IndentedTextWriter 写入的内容，最终将会写入到 StringBuilder 里面，初始化的代码如下

```csharp
var stringBuilder = new StringBuilder();
var stringWriter = new StringWriter(stringBuilder);
var indentedTextWriter = new IndentedTextWriter(stringWriter, " ");
```

在初始化 IndentedTextWriter 时，可选传入 `tabString` 参数，此参数表示用来表示缩进的字符串。例如缩进等级为 1 时，将在每个行之前写入 1 个传入的 `tabString` 参数字符串，如上文代码，就是写入一个空格。如果自己传入其他的参数，例如两个空格，那就表示一个缩进等级写入两个空格。如自己传入 `-` 就表示每个缩进传入的是 `-` 字符

通过 `IndentedTextWriter.Indent` 属性可以控制当前的缩进等级，例如以下代码设置缩进为 2 缩进等级

```csharp
indentedTextWriter.Indent = 2;
```

试试在缩进前后写入内容，看看缩进对写入内容的影响

```csharp
indentedTextWriter.WriteLine("Hello");
indentedTextWriter.WriteLine("Hello");
indentedTextWriter.Indent = 2;

indentedTextWriter.WriteLine("Hello");
indentedTextWriter.WriteLine("Hello");

Console.WriteLine(stringBuilder.ToString());
```

输出内容如下

```
Hello
Hello
  Hello
  Hello
```

通过加等和减等控制缩进等级，可以比较好的实现花括号的缩进，如以下代码

```csharp
var stringBuilder = new StringBuilder();
var stringWriter = new StringWriter(stringBuilder);
var indentedTextWriter = new IndentedTextWriter(stringWriter, " ");
indentedTextWriter.WriteLine("Hello");
indentedTextWriter.WriteLine("Hello");
indentedTextWriter.Indent = 2;

indentedTextWriter.WriteLine("Hello");
indentedTextWriter.WriteLine("Hello");

indentedTextWriter.Indent += 2;
indentedTextWriter.WriteLine("{");

indentedTextWriter.Indent += 4;
indentedTextWriter.WriteLine("Hello");
indentedTextWriter.WriteLine("Hello");

indentedTextWriter.Indent -= 2;
indentedTextWriter.WriteLine("Hello");
indentedTextWriter.Indent -= 2;

indentedTextWriter.WriteLine("}");

indentedTextWriter.Indent -= 2;

Console.WriteLine(stringBuilder.ToString());
```


输出内容如下

```csharp
Hello
Hello
  Hello
  Hello
    {
        Hello
        Hello
      Hello
    }
```

由于 IndentedTextWriter 的构造函数可以让缩进采用除了空格之外的其他字符串内容，这就可以让大家更加开森，可以将 IndentedTextWriter 用来除代码之外的其他生成内容里

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/e54d3f45986ff8200d8601cd8dc0bedc81924d75/HoyebenawlerWegemnardicheba/TestIndentedTextWriter) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e54d3f45986ff8200d8601cd8dc0bedc81924d75/HoyebenawlerWegemnardicheba/TestIndentedTextWriter) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e54d3f45986ff8200d8601cd8dc0bedc81924d75
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e54d3f45986ff8200d8601cd8dc0bedc81924d75
```

获取代码之后，进入 HoyebenawlerWegemnardicheba 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
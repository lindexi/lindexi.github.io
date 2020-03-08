# dotnet core 编程规范

本文实际只是翻译  .NET Core foundational libraries 官方文档的编码风格。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


在 [.NET Core foundational libraries](https://github.com/dotnet/corefx )项目使用的编程规范默认就是 VisualStudio 默认样式。

## 花括号

花括号使用  Allman style 风格，所有的花括号在新的一行开始。

关于 Allman style 就是如下，花括号放在下一行，而且花括号不进行缩进，花括号里面的代码缩进。

```csharp
while (x == y)
{
    something();
    somethingelse();
}
```

但是对于单行的语句，可以不添加花括号，但是请把它放在同一行

```csharp
          if (xx == null) xx = new Foo();
```

单行语句同样建议添加花括号避免[苹果](https://www.imperialviolet.org/2014/02/22/applebug.html )的bug

如果不添加花括号，一定不要在嵌套时使用。

## 空格

使用 4 个空格作为一个缩进，而不是使用 tab 

## 字段

所有的字段使用`_camelCase`风格，而且尽可能使用`readonly `。在 internal 和 private 访问的字段添加 `_` 前缀，对于静态的字段添加`s_`前缀，对于线程静态的字段添加`t_`前缀。如果使用了静态的字段而且可以设置`readonly `，需要把`readonly `放在`static`后面。

最好不要公开字段，如果需要公开那么使用PascalCasing风格而且不添加前缀。

我不是很赞同对字段的风格，但是因为本文是翻译，我就没有添加自己的理解。

## 限定

不要添加 `this.` ，完全是没必要的。

我也不是很赞同这个，因为如果使用扩展方法，需要使用`this.`

## 访问

显示指定访问，即使访问是默认的，如`private string _foo` ，而不是`string _foo`。访问需要放在最前面，如`public abstract`，不建议`abstract public`

默认的访问是可能修改的

## 命名空间

对于引用需要放在文件的开头。

## 空行

禁止超过一行的空白，不可以使用两行空白。

不要使用两行分割两个类型。

## 多余空格

禁止在一行结束添加多余空格。通过 VisualStudio 的查看空格(ctrl+E,S)可以看到空格

## 参照现有

如果一个文件在格式规范定义之前已经有自己的规范，尽可能参照文件现有的规范而不是去修改他。

新的修改参照现有的文件的格式。

我一点都不赞同。

## 明显的类型

只能在明显可以知道对象类型可以使用 var ，如`var stream = new FileStream(...)` 。在无法明显知道对象类型不可以使用 var ，如`var stream = OpenStandardInput()` 。

## 关键字

使用语言的关键字代替 BCL 类型，如使用 `int, string, float`代替`Int32`，`String`，`Single`

## 常量命名

对于所有的常量，包括静态只读，使用 `PascalCasing` 风格。如果需要使用 C++ 或其他代码对于命名有要求才可以使用其他的方法。

## 变量名字符串

如果使用变量名的常量，必须使用 `nameof` 关键字。

## 字段结构

字段必须放在一个类声明的最前面。

我一点都不赞成，因为谁会去管一个类的字段，我关心的是构造函数和公开属性、函数。

## 其他字符

如果在代码使用了 非 ascii 的字符，那么需要把他转换为 `\uxx` 方式，这样可以解决一些非 ascii 因为编译器分析的问题。

当然上面的要求只是对于编译的代码

## Label

使用 Label 缩进一个小于当前缩进的标签

原文：[coding-style.md](https://github.com/dotnet/corefx/blob/master/Documentation/coding-guidelines/coding-style.md )

![](http://image.acmx.xyz/lindexi%2F2018571237288065.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

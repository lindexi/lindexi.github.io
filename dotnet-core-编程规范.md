
# dotnet core 编程规范

本文实质为翻译 .NET Core foundational libraries 官方文档的编码风格。

<!--more-->


<!-- CreateTime:2018/8/10 19:16:52 -->

此规范和 VisualStudio 默认约束相同，绝大部分 C# .NET 系项目都会遵循此规范

<!-- 在 [.NET Core foundational libraries](https://github.com/dotnet/runtime )项目使用的编程规范默认就是 VisualStudio 默认样式。 -->
原文：[coding-style.md](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md )

## 花括号

花括号使用 Allman style 风格，所有的花括号在新的一行开始。

关于 Allman style 的简要描述如下：花括号放在下一行，而且花括号不进行缩进，花括号里面的代码缩进。

```csharp
while (x == y)
{
    something();
    somethingelse();
}
```

对于单行语句块可以没有大括号，但是该块必须在单独的一行中正确缩进，且不能嵌套其他除 `using` 外大括号语句块之外

```csharp
          if (xx == null)
            xx = new Foo(); // 尽管允许，但建议尽量添加花括号
```

注： 单行语句同样建议添加花括号避免[苹果](https://www.imperialviolet.org/2014/02/22/applebug.html )的bug

<!-- 如果不添加花括号，一定不要在嵌套时使用。 -->

只有在每个与 `if / else if /.../ else` 复合语句相关的代码块都放在单行上时，才可以省略大括号

注： 我建议能加花括号的，全都加花括号。即使是单行

不要使用一句的单行形式，如 `if (source == null) throw new ArgumentNullException("source");` 的写法

注： 和规范不同的是，我允许使用一句的单行形式。我认为单行和多行都不直观，我期望添加花括号。无论如何，使用大括号总是被接受，总是被推荐的

注： 关于单行语句，是存在很多争议的，也有过一些优化提案。详细请参阅：

- <https://github.com/dotnet/csharplang/discussions/1493>
- <https://github.com/dotnet/csharplang/discussions/1785>
- <https://github.com/dotnet/roslyn/issues/11562>

## 空格

使用 4 个空格作为一个缩进，而不是使用 tab 的宽度

## 字段

所有的字段使用 `_camelCase` 风格，而且尽可能使用 `readonly ` 标记只读。只要是语法语义逻辑上是只读，就标记 `readonly ` 只读。使用了 internal 或 private 访问权限的字段应该添加 `_` 前缀，对于静态的字段添加 `s_` 前缀，对于线程静态的字段添加`t_`前缀。如果使用了静态的字段而且可以设置 `readonly ` 只读的，需要把`readonly`放在`static`后面，即 `static readonly`，而非 `readonly static`

最好不要公开字段，如果需要公开则请使用 `PascalCasing` 命名风格而且不添加前缀。

注： 我不是很赞同对字段的添加前缀风格。我认为字段只添加 `_` 下划线前缀即可，不要添加 `s_` 或 `t_` 前缀，核心观点在于代码语义随时变更，不能确保开发者同步更改命名

附：

- camelCase 风格：单词之间使用首字符大写分割，首个字符使用小写
- PascalCasing 风格：单词之间使用首字符大写分割，首个字符使用大写

## 限定

不要添加 `this.` ，完全是没必要的。除非是扩展方法这种必须添加 `this.` 的情况

## 访问

显式指定访问权限关键字，即使访问权限是默认的，如`private string _foo` ，而不是`string _foo`。访问权限关键字需要放在最前面，如`public abstract`，不建议`abstract public`

默认的访问是可能修改的，当前的行为只是默认 .NET 的规定

## 命名空间

对于引用需要放在文件的开头。

如无特殊需求，对命名空间进行排序更佳，且排序时让 System 开头的命名空间放在前面

## 空行

禁止超过一行的空白，不可以使用两行空白。

不要使用两行分割两个类型。

## 多余空格

禁止在一行结束添加多余空格。通过 VisualStudio 的查看空格(ctrl+E,S)可以看到空格

## 参照现有规范

如果一个文件在格式规范定义之前已经有自己的规范，尽可能参照文件现有的规范而不是去修改他。

新的修改参照现有的文件的格式。

我一点都不赞同。我认为应该来一个大的更改，只更改样式和规范。不要保持旧项目奇怪的写法

## 明显的类型

只能在明显可以知道对象类型时，才可以使用 var 关键字，如 `var stream = new FileStream(...)`  。在无法明显知道对象类型时，不可以使用 var 关键字，如`var stream = OpenStandardInput()` 。

类似地，省略类型的 `new()` 也仅当左侧明确命名类型的情况下使用。如在局部变量或字段定义语句中使用，如 `FileStream stream = new(...);` 。但如左侧无明确命名类型时，不应使用 `new()` 语法，如以下情况

```csharp
FileStream stream = xx;
... // 经过某些逻辑
stream = new(...); // 禁止此写法。因为 new() 的左侧无明确的类型
```

## 关键字

使用语言的关键字代替 BCL 类型，如使用 `int, string, float`代替`Int32`，`String`，`Single`

## 常量命名

对于所有的常量，包括静态只读，使用 `PascalCasing` 风格。如果需要使用 C++ 或其他代码对于命名有要求时才可以使用其他的命名方法

## 变量名字符串

如果使用变量名的常量，必须使用 `nameof` 关键字。能够使用 `nameof` 关键字的地方，尽量使用 `nameof` 关键字

## 字段结构

字段必须放在一个类声明的最前面。

注： 我一点都不赞成，因为谁会去管一个类的字段，我关心的是构造函数和公开属性、函数。

## 其他字符

如果在代码使用了 非 ascii 的字符，那么需要把他转换为 `\uxx` 方式，这样可以解决一些非 ascii 因为编译器分析的问题。

当然上面的要求只是针对于参与编译的代码，不参与构建的部分，如注释、内建字符串等，不在此要求范围内

## Label

使用 Label 缩进一个小于当前缩进的标签

## 主构造函数

非 record 记录类型的主构造函数参数应该像普通参数一样命名，使用`camelCase`格式，并且不应以`_`前缀。例如，应使用 `public ObservableLinkedList(IEnumerable<T> items)` 而不是 `public ObservableLinkedList(IEnumerable<T> _items)`

如果使用了主构造函数的非 record 记录类型的代码不足够小，不能一眼看出主构造函数参数使用位置，或可能主构造函数参数与局部变量混淆。那么应该新建一个使用 `_` 下划线前缀的字段，将主构造函数参数分配给此字段，且在代码逻辑中使用字段而不是主构造函数参数，如

```csharp
private readonly IEnumerable<T> _items = item;
```

![](http://cdn.lindexi.site/lindexi%2F2018571237288065.jpg)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
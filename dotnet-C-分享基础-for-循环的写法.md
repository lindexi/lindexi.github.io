
# dotnet C# 分享基础 for 循环的写法

本文将来和大家聊聊在 dotnet C# 里面的基础的 for 循环语法的写法

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

使用 for 作为循环的写法是在编程里面常用的代码写法。比如说我有一个名为 Foo 的类型，这个类型提供了一个名为 GetCount 的方法，这个方法可以缓慢的获取 Foo 里面的一个代表数量的值。且在 Foo 类型里面提供了索引器，可以根据传入的下标序号返回一个字符串，例子代码如下

```csharp
class Foo
{
    public string this[int index]
    {
        get
        {
           return "Hello" + index;
        }
    }

    public int GetCount()
    {
        // 模拟一个耗时的操作
        Thread.Sleep(100);

        return 100;
    }
}
```

假定现在的需求是需要遍历 Foo 返回的数量，通过 Foo 的索引器获取其值。那么最简单的写法大概如下

```csharp
var foo = new Foo();

for (int i = 0; i < foo.GetCount(); i++)
{
    var value = foo[i];
    _ = value;
}
```

如以上代码所示，在 for 循环里面，通过在条件语句里面的 `i < foo.GetCount()` 判断是否超过了 Foo 数量范围，从而完成循环条件判断

以上代码的 `_ = value;` 只是用于示例表示取出了 value 值，模拟拿出去用。这里的 `_ =` 表示的是右值不受关注，不受使用，只是一个让编译器开森的代码，让编译器不要警告说 value 局部变量没地方使用而已

由于 Foo 的 GetCount 方法是耗时的，以上代码将会在每次循环判断条件语句里面调用 GetCount 方法。无疑这将会是低效率的

咱又提前知道了 GetCount 方法的返回值不会在循环过程中变更，一个比较好的方法就是提前先取出 GetCount 方法的返回值，将返回值存放到局部变量里面，如此即可减少此方法的调用

```csharp
var fooCount = foo.GetCount();
for (int i = 0; i < fooCount; i++)
{
    var value = foo[i];
    _ = value;
}
```

如上述代码所示，可以看到在 for 分支判断语句里面只使用 `i < fooCount` 判断逻辑，不再需要每次判断时都调用 GetCount 方法，可以提升不少的性能

然而以上代码却多引入了一行 `var fooCount = foo.GetCount();` 代码，代码行数上比不过最初的代码。在不关注耗时或性能的代码上，这样看起来最初的循环代码会比上述代码更加简短

如果只是为了减少代码行，那可以将 `var fooCount = foo.GetCount();` 放入到 for 循环的初始化表达式语句里面编写，如下面代码

```csharp
for (int i = 0, count = foo.GetCount(); i < count; i++)
{
    var value = foo[i];
    _ = value;
}
```

以上的 for 循环语句的初始化表达式的代码就是 `int i = 0, count = foo.GetCount()` 同时初始化 i 和 count 两个局部变量

尽管以上两个代码看起来行数不相同，但从底层上没有差别

以上例子只是告诉大家不要有编程的习惯，误以为 for 循环的初始化语句只能初始化循环的索引局部变量。其实在 for 循环的初始化语句里面可以放入你开森的逻辑

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f005404efbf000f19880fad4f03cc2c6c242e967/Workbench/KerewaykeehabalheaHebalhereda) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f005404efbf000f19880fad4f03cc2c6c242e967/Workbench/KerewaykeehabalheaHebalhereda) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f005404efbf000f19880fad4f03cc2c6c242e967
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f005404efbf000f19880fad4f03cc2c6c242e967
```

获取代码之后，进入 Workbench/KerewaykeehabalheaHebalhereda 文件夹，即可获取到源代码

更多基础语法请看 <https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/statements/iteration-statements>

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
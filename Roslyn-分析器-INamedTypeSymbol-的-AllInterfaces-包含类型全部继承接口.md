
# Roslyn 分析器 INamedTypeSymbol 的 AllInterfaces 包含类型全部继承接口

本文将记录和测试 Microsoft.CodeAnalysis.INamedTypeSymbol 的 AllInterfaces 属性能获取的指定类型的接口范围

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

经过测试，发现 AllInterfaces 属性如其注释所述，可以获取类型继承的全部接口，包括所继承的基类型的接口，接口本身继承的接口

如以下测试代码所示

```csharp
public interface IFoo
{
}

public interface IF1 : IFoo
{
}

public interface IF2 : IF1
{
}

public class Foo : IF2
{
}

public interface IF3
{

}

public class F1 : Foo, IF3
{
}
```

拿到 F1 的 INamedTypeSymbol 之后，获取 AllInterfaces 属性，可以获取到 IF2;IF1;IFoo;IF3 这些接口信息。通过上面代码可见 F1 类型只继承了 Foo 类型和 IF3 接口，但在获取 AllInterfaces 属性时，语义可以返回真实所继承的全部接口信息，和上层编写代码运行时效果一致

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0d5a5ce70d5780ccb0ef7fe158335d2e97a71bcb/Roslyn/WejeajaihurleyereRearchicobairnurjeyu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/0d5a5ce70d5780ccb0ef7fe158335d2e97a71bcb/Roslyn/WejeajaihurleyereRearchicobairnurjeyu) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0d5a5ce70d5780ccb0ef7fe158335d2e97a71bcb
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0d5a5ce70d5780ccb0ef7fe158335d2e97a71bcb
```

获取代码之后，进入 Roslyn/WejeajaihurleyereRearchicobairnurjeyu 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
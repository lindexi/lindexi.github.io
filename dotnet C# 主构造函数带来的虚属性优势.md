# dotnet C# 主构造函数带来的虚属性优势

在基类里面使用虚属性，在构造函数调用抽象或虚方法时，一个令开发者烦恼的点在于调用的抽象或虚方法可能访问到子类还没赋值的属性，导致出现空异常

<!--more-->
<!-- CreateTime:2025/12/03 07:15:14 -->

<!-- 发布 -->
<!-- 博客 -->

试试看以下代码，看看 F2 对象被构造出来的时候会发生什么事情，在 D 方法里面是否能够拿到 Foo 属性的值

```csharp
class Foo
{
}

class F1
{
    public F1(Foo foo)
    {
        Foo = foo;
        D();
    }

    public virtual Foo Foo { get; }

    public virtual void D()
    {
    }
}

class F2 : F1
{
    public F2(Foo foo) : base(foo)
    {
        Foo = foo;
    }

    public override Foo Foo { get; }

    public override void D()
    {
        var f = Foo;
    }
}
```

按照类型的构造函数的顺序，是先执行基类的构造函数，接着再执行子类的构造函数。由此可以知道，只有在子类的构造函数被调用的时候，才会给 Foo 属性赋值。然而 D 方法却在基类的构造函数里面调用，这就意味着此时 F2 里的 D 方法拿到的 Foo 属性必定会是空

以上就是比较经典的在基类调用虚或抽象的方法或属性时会遇到的困难点

主构造函数从语法层面上很好地解决了此问题，将以上的 F2 代码的构造函数变更为主构造函数，如以下代码所示

```csharp
class F2(Foo foo) : F1(foo)
{
    public override Foo Foo => foo;

    public override void D()
    {
        var f = Foo;
    }
}
```

此时创建 F2 对象时，从 F1 构造函数调用的 F2 的 D 方法拿到的 Foo 属性将不是空。这是因为 Foo 属性是一个没有后备字段的属性，只是从主构造函数捕获的 `foo` 变量进行返回。由于 `foo` 是在主构造里面捕获的，属于语法确保类里面可用且有的变量，这就意味着无需等待 F2 执行构造函数即可拿到值

这就意味着主构造函数和显式构造函数之间不是等价变换的，这一点还请大家在做代码变更的时候着重思考是否主构造函数已经是被某个虚或抽象的属性给捕获，且这个属性还可能被基类的构造函数访问到。如果如何以上条件，则主构造函数不能等价修改为显式构造函数写法

也许有伙伴好奇为什么主构造函数能从语法层面上带来这一点的优势，里面的魔法是什么。其实这里面没有什么魔法，只是一个调用顺序的问题，从反编的 IL 可以获得答案。当然了 IL 代码还是比较人类不友好的，我下面贴出来从 IL 转换为低级 C# 代码

```csharp
   internal class F2 : F1
   {
     [CompilerGenerated]
     [DebuggerBrowsable(DebuggerBrowsableState.Never)]
     private Foo <foo>P; // 这就是主构造函数捕获的变量对应的字段。这是构建的时候生成的。这里的变量名是 `<foo>P` 包含两个尖括号的内容。在 IL 里面尖括号是合法的变量名字符，只是在 C# 里面不被允许。于是充分利用此规则，就可以生成不会在开发者代码被调用的字段
   
     public F2(Foo foo)
     {
       this.<foo>P = foo;
       base..ctor(this.<foo>P); // 这里的 `.ctor` 就是构造函数的意思。这里的 `base..ctor` 是 `base.` 和 `.ctor` 的意思，即调用 base 基类的 `.ctor` 构造函数
     }
   
     public override Foo Foo
     {
       get
       {
         return this.<foo>P;
       }
     }
   
     public override void D()
     {
       Foo f = this.Foo;
     }
   }
```

可以看到在主构造函数里面写的 `foo` 变量，被捕获为 `<foo>P` 字段。且生成的构造函数的代码调用顺序也十分有趣，先将 `foo` 赋值给到 `<foo>P` 字段，再将字段传入到基类的构造函数。于是在这个过程里面，就可以确保 `<foo>P` 字段一定在基类的构造函数调用之前被赋值。进而让用到主构造函数捕获的变量的代码逻辑可以确保实际构建出来的代码是从一开始赋值的字段获取的，如此就可以解决基类直接或间接访问虚或抽象属性时可能的空问题

必须说明的是，我对主构造函数这个语法是有一些原则的。只有在类足够小的时候，我才会考虑主构造函数。这是因为主构造函数会捕获传入参数变量的缘故，导致在有一定规模的类里面，会和局部变量混淆冲突，影响我的代码阅读。只有在很小的类里面，写主构造函数能够实际地减少代码量的时候，才会考虑使用

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/992ea5dea2fe5711b286ded3454075e223c34146/Workbench/WonewheajeaNelbaylairreda) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/992ea5dea2fe5711b286ded3454075e223c34146/Workbench/WonewheajeaNelbaylairreda) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 992ea5dea2fe5711b286ded3454075e223c34146
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 992ea5dea2fe5711b286ded3454075e223c34146
```

获取代码之后，进入 Workbench/WonewheajeaNelbaylairreda 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
# C# ValueTuple 原理

本文告诉大家一些 ValueTuple 的原理，避免在使用出现和期望不相同的值。ValueTuple 是 C# 7 的语法糖，如果使用的 .net Framework 是 4.7 以前，那么需要使用 Nuget 安装`System.ValueTuple`

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->

<div id="toc"></div>

<!-- 标签：C#，原理 -->

虽然 ValueTuple 的很好用，但是需要知道他有两个地方都是在用的时候需要知道他原理。如果不知道原理，可能就发现代码和预期不相同

## json 转换

先创建一个项目，然后安装 Json 解析，使用下面的代码，在运行之前，先猜一下，下面的代码会出现什么

```csharp
            var foo = (name: "lindexi", site: "blog.csdn.net/lindexi_gd");
            var str = JsonConvert.SerializeObject(foo);
```

实际上输出的是 `{"Item1":"lindexi","Item2":"blog.csdn.net/lindexi_gd"}`

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201822162054.jpg)

那么刚才的命名在哪？

如果想知道，那么请看 ValueTuple 的原理

## 原理

先来写一段代码，编译之后对他反编译，看一下他是怎么做的

```csharp
        static void Main(string[] args)
        {
            var foo = Foo();
            var str = JsonConvert.SerializeObject(foo);
            Console.WriteLine(str);
        }

        static (string name, string site) Foo()
        {
            return (name: "lindexi", site: "blog.csdn.net/lindexi_gd");
        }
```

不需要安装反编译软件，可以使用这个[网站](https://sharplab.io/)拿到反编译

可以看到Foo被编译为 TupleElementNames 特性的两个字符串

```csharp
    [return: TupleElementNames(new string[]
    {
        "name",
        "site"
    })]
    private static ValueTuple<string, string> Foo()
    {
        return new ValueTuple<string, string>("lindexi", "blog.csdn.net/lindexi_gd");
    }
```

所以实际上代码是 `ValueTuple<string, string> ` 不是刚才定义的代码，只是通过 TupleElementNames 让编译器知道值，所以是语法糖。

IL 代码是 

```csharp
private hidebysig static valuetype [mscorlib]System.ValueTuple`2<string, string> 
    Foo() cil managed 
  {
    .param [0] 
    .custom instance void [mscorlib]System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[]) 
      = (
        01 00 02 00 00 00 04 6e 61 6d 65 04 73 69 74 65 // .......name.site 这里就是 return: TupleElementNames 的命名
        00 00                                           // ..
      )
    .maxstack 2
    .locals init (
      [0] valuetype [mscorlib]System.ValueTuple`2<string, string> V_0
    )

    // [20 9 - 20 10]
    IL_0000: nop          

    // [21 13 - 21 72]
    IL_0001: ldstr        "lindexi"
    IL_0006: ldstr        "blog.csdn.net/lindexi_gd"
    IL_000b: newobj       instance void valuetype [mscorlib]System.ValueTuple`2<string, string>::.ctor(!0/*string*/, !1/*string*/)
    IL_0010: stloc.0      // V_0
    IL_0011: br.s         IL_0013

    // [22 9 - 22 10]
    IL_0013: ldloc.0      // V_0
    IL_0014: ret          

  }
```

这个特性只有编译器可以用，不可以在代码使用。

在上面的解释，实际上 IL 不知道存在定义的命名，所以不可以通过这个方法获得值。

## 动态类型获得值

如果希望使用动态类型获得值，那么下面的代码实际上会运行出现异常

```csharp
        static void Main(string[] args)
        {
            dynamic foo = Foo();
            Console.WriteLine(foo.name);
        }

        static (string name, string site) Foo()
        {
            return (name: "lindexi", site: "blog.csdn.net/lindexi_gd");
        }
```


运行出现 RuntimeBinderException 异常，因为没有发现 `name` 属性

实际上对比下面匿名类，也就是很差不多写法。

```csharp
        dynamic foo = new { name = "lindexi", site = "blog.csdn.net/lindexi_gd" };
            Console.WriteLine(foo.name);
```

运行是可以的，所以在使用动态类型，请不要使用 ValueTuple ，如果需要使用，那么请知道有存在找不到变量异常，而且是在运行才出现异常。

## 性能提升

如果使用 ValueTuple 编程会有一些优点，性能是其中之一。而且对于异步编程，使用 ValueTuple 可以继续使用 await 的方法。

假如有一个方法需要返回 5 个参数，那么以前的做法有三个方法，第一个方法是使用 out 的方法，第二个方法是使用 Tuple ，第三个方法是定义一个临时的类。

如果使用了 out 的方法，那么这个方法就不可以继续使用异步 await 的方法，因为 await 需要做出状态机，参见我写的await原理。如果使用 Tuple ，或这定义一个临时的类，就会出现性能的问题。

从上面的原理，已经告诉大家，ValueTuple 是值类型，而 Tuple 或定义的一个类不是值类型。编译器的优化是让 ValueTuple 分配在栈，对于普通的类分配在堆空间。如果一个类分配到堆空间，那么就需要使用垃圾回收才可以清理空间。而分配到栈就不需要使用垃圾回收，使用完成就清空栈，效率比堆空间大。

但是使用栈空间需要注意，栈空间是很小的，如果使用了大量栈空间可能会出现堆栈gg。因为考虑到部分刚入门的小伙伴，所以我就需要多说一些，上面说的 ValueTuple 使用了栈空间需要小心栈空间不足，和你存放的值的关系不大，而是和定义的 ValueTuple 数量有关，这个数量是非常大的。但是在递归方法中，本来是刚好空间足够的，在使用了 ValueTuple 可能就不够了。

使用 ValueTuple 可以继续使用异步，而且不需要垃圾回收，性能比Tuple高，所以建议在多返回参数使用 ValueTuple，而不是定义一个类。

## 其他需要知道的

不要随便定义一个看不懂的值

实际上下面的代码，编译是可以通过

```csharp
(int x, (int y, (float a, float b))[] c) f1
```

但是这个值，在看的时候，几乎说不出他的属性

第二个需要知道的，ValueTuple 是值类型，所以他的默认值不是 null 而是 `default(xx)`，在C# 7.2 支持使用关键字，所以不需要去写 `defalut(xx,xx)`

关于 ValueTuple 变量名的定义也是很难说的，有的小伙伴觉得需要使用 Axx 的方式命名，但是很多小伙伴觉得使用 aaBa 的命名更好，所以暂时对于他的命名使用 aaBa 的方法，大家觉得什么方式好请告诉我

参见：[ Exploring Tuples as a Library Author](http://blog.marcgravell.com/2017/04/exploring-tuples-as-library-author.html )

[C# 7: Dynamic types and Reflection cannot access Tuple fields by name](https://www.danielcrabtree.com/blog/99/c-sharp-7-dynamic-types-and-reflection-cannot-access-tuple-fields-by-name )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
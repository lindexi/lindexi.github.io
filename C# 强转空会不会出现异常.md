# C# 强转空会不会出现异常

有小伙伴问我强转 null 会不会出现异常，我告诉他，如果是引用类型那么不会，如果是值类型，那么会出现空异常

<!--more-->
<!-- CreateTime:2019/10/31 8:53:06 -->


如果是引用类型，只要是空类型，是支持随意转换，如下面代码，这是可以运行

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            Program p = null;

            object obj = p;

            Foo foo = (Foo) obj;
        }
    }

    class Foo
    {

    }
```

如果使用值类型转换，那么将会出现空异常，例如我定义一个枚举

```csharp
    enum NerefiweakawBejairlalhu
    {
        
    }
```

下面代码运行的时候会提示

```csharp
System.NullReferenceException:“Object reference not set to an instance of an object.”
```

也就是如果你看到了泛型的转换，请确定泛型不会传入值类型

```csharp
    class Foo<T>
    {
        public void Cast(object obj)
        {
            var foo = (T) obj;
        }
    }
```

上面代码如果用户传入了值类型，例如 枚举 作为泛型，那么调用 Cast 传入空的值，将会提示对象为空，所以在使用泛型转换的时候，可能强转为空

如果此时将强转换为 as 关键字，将会提示 由于类型参数“T”既没有类类型约束也没有“class”约束，因此不能与“as”运算符一起使用

如果要给泛型约束只能给引用类型用，那么请加上 class 条件

```csharp
    class Foo<T> where T : class
```

如果需要给值类型用，请使用下面代码

```csharp
class Foo<T> where T : struct
```

如果看到了一个 obj 强转一个值类型，那么在 obj 为空的时候出现异常，推荐的方法是通过 is 关键字，在 C# 7.0 的时候可以使用 is 匹配，请看下面代码

```csharp
    class Foo<T> where T : struct
    {
        public void Cast(object obj)
        {
            if (obj is T t)
            {
                var foo = t;
            }
        }
    }
```

当前，请记得加上 else 提示用户传入的值不能强转传入的类型

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

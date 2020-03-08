# C# 匹配可空变量

在 C# 7.0 的时候提供更好用的模式匹配方法，支持通过 is 直接转换对应的类，但是如果是尝试转换可空的对象，那么将会提示无法编译，或转换失败

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 [C# 7.0](https://blog.lindexi.com/post/C-7.0.html) 的 is 转换是十分好用的功能，例如我写了一个 Foo 函数，支持将传入的参数转换

```csharp
        private static void Foo<T>(object o)
        {
            if(o is T t)
            {

            }
        }
```

此时会判断如果参数属于泛型 T 那么就转换同时拿到对象 t 用于在判断里面用

但是有小伙伴说他传入了一个 `bool?` 问我在什么时候才能进入判断

于是我就尝试了下面的代码

```csharp
            Foo<bool?>(null);
            Foo<bool?>(true);
```

此时发现传入 `null` 的时候不会进入判断，同时传入 true 的时候可以进入判断

也就是对于 `if (null is bool? b)` 将会一直返回 false 同时这段代码也编译不通过，如果我将可空包装一下会如何

```csharp
            object o = null;

            if (o is Nullable<bool> b)
            {

            }
```

其实上面代码也是编译不通过的，会看到提示在模式匹配里面使用可以为 null 的类型`bool?`是非法的；请改用基础类型`bool` 也就是模式匹配里面对于空的判断是认为小伙伴不能这样写

这个用法和 as 有一些不同

```csharp
var b = null as bool?;
```

上面代码可以计算出一个为空的 `bool?` 但是在模式匹配里面是不进入判断

也就是在模式匹配里面其实不包含可空

这个问题有[Blue](https://github.com/Blue0500)小伙伴在 github 上的[roslyn #20156](https://github.com/dotnet/roslyn/issues/20156 ) 提出，他的问题翻译出来大概是这样

在进行可空的模式匹配的时候，将会编译出错，如果使用 `Nullable<int>` 将提示错误 `CS8116` 编译失败。使用 `int?` 将会提示 `CS1003` `CS1525` `CS0103` 编译失败

这里需要了解一下运算对于 is 的做法，对于 `int?` 其实你会看到格式化的时候是 `int ?` 中间有一个空格

```csharp
            object o = null;

            if (o is bool ? b)
            {

            }
```

这是因为这句表达式编译出来的是 `if ((o is bool) ? b)` 这里还缺少的就是后面的值

```csharp
 if (o is bool ? true : false)
``` 

这里的 `o is bool?` 作为的是运算符 `(o is bool) ? 如果o是bool 的时候的值: 如果不是的时候的值` 所以提示的无法编译就是找不到定义的变量和表达式为 false 的变量

那么现在尝试做一道题

```csharp
    class B
    {
        public static int operator &(B left, B right) => 1;
        public static int operator >(B left, B right) => 2;
        public static int operator <(B left, B right) => 3;

        public static int operator &(bool left, B right) => 5;
        public static int operator >(bool left, B right) => 6;
        public static int operator <(bool left, B right) => 7;
    }

         static void Main(string[] args)
         {
             object a = null;
             B c = null;
             Console.WriteLine(a is B & c);
             Console.WriteLine(a is B > c);
             Console.WriteLine(a is B < c);
         }
```

请问上面代码输出多少

是将 `a is B` 的值作为 bool 转入计算还是将 `a is B b` 这个隐藏的 b 传入计算

请看 [int? 竟然真的可以是 null](https://blog.walterlv.com/post/how-to-identify-a-nullable-type.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

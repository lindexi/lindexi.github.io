# C＃ 谁改了我的代码

本文告诉大家一个特殊的做法，可以修改一个字符串常量

<!--more-->
<!-- CreateTime:2018/3/31 21:15:03 -->


我们来写一个简单的程序，把一个常量字符串输出

```csharp
        private const string str = "lindexi";
        static void Main(string[] args)
        {
            Foo();
            Console.WriteLine(str);
        }
```

其中的 Foo 是其他的函数，大家可以猜到输出是 lindexi ，但是，实际上把Foo调用函数添加之后，输出是 Lindexi 被大写了。那么这时 Foo 做了什么？

Foo 做的就是
[C＃ 字符串首字符大写](https://lindexi.oschina.io/lindexi/post/C-%E5%AD%97%E7%AC%A6%E4%B8%B2%E9%A6%96%E5%AD%97%E7%AC%A6%E5%A4%A7%E5%86%99.html )

```csharp
        public static unsafe void Foo()
        {
            fixed (char* ptr = str)
            {
                *ptr = char.ToUpper(*ptr);
            }
        }
```

虽然出现了问题，但是找到问题很简单，如果这时需要做一个安全有关的。让别人看到源代码也不知道怎么使用，那么就可以使用这个科技，下面就是显示技术的时候

我把 Foo 做一些修改，把 str 变量去掉，这样大家就难以通过搜索变量引用找到了这个函数。但是我在其他的某个地方使用了这个常量字符串，于是就把上面的 str 修改为 "lindexi" 。大家也许会想，这是两个变量，对他做什么修改也不会对之前的 str 有什么影响。实际上，请跑一下下面的代码。

```csharp
        public static unsafe void Foo()
        {
            fixed (char* ptr = "lindexi")
            {
                *ptr = char.ToUpper(*ptr);
            }
        }
```

这时输出 str 结果是 Lindexi ，因为编译器把相同的常量视为同一个地址，这样修改一个地方的常量就可以修改其他地方的。所以可以写的是一个常量，实际上这个常量在另一个地方被修改。

如果我代码很多，在某个地方使用了反射，反射一个方法，这个方法是修改一个常量的值，常量是写自己写的，没有引用。这时可以发现代码执行就可以更改之前的字符串值。实际上不只字符串，其它的常量也可以修改。多使用这些技术，可以让看代码的人成为强大的杀人狂。

这个方法是不推荐在一般情况使用，因为谁都不能说没有其他地方使用一样的字符串。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativeco
mmons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
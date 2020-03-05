# dotnet 非泛型 类型 System.Collections.IEnumerable 不能与类型实参一起使用

如果在开发的时候遇到非泛型 类型“IEnumerable”不能与类型参数一起使用，那么就是变量的命名空间没弄对

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->


在 dotnet 里面有 `System.Collections.IEnumerable` 和 `System.Collections.Generic.IEnumerable<>` 两个不同的类，带泛型的需要在 `System.Collections.Generic` 命名空间找到

如果是写了 `System.Collections.IEnumerable<Foo>` 那么请修改代码里面的命名空间 `System.Collections.Generic.IEnumerable<Foo>` 就可以通过编译

如果是使用 `IEnumerable<Foo>` 提示 不能与类型实参一起使用，那么只需要添加 using 就可以

```csharp
using System.Collections.Generic;
```

除了 IEnumerable 对于 IEnumerator 也一样，如果遇到非泛型 类型“System.Collections.IEnumerator”不能与类型实参一起使用，那么看代码里面是通过 `System.Collections.IEnumerator<Foo>` 还是 `IEnumerator<Foo>` 可以选择添加命名空间还是修改

![](https://i.loli.net/2019/04/14/5cb29f897d199.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

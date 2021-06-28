# dotnet C# 使用 EqualityComparer 提升泛型值类型相等判断性能

本文也叫跟着 Stephen Toub 大佬学性能优化系列，这是我从 Stephen Toub 大佬给 WPF 框架做性能优化学到的知识，通过 EqualityComparer 静态类的相等方法来优化值类型相等判断性能

<!--more-->
<!-- CreateTime:2021/6/25 19:30:39 -->

<!-- 发布 -->

在一些泛型类型里面，需要进行值相等判断，此时默认就是使用 Equals 方法，如下面代码

```csharp
public override bool Contains(T value)
{
    return _loneEntry.Equals(value);
}
```

还请忽略上面代码的 `_loneEntry` 字段，但是以上的代码调用的 Equals 方法的参数是 object 类型，也就是调用 Equals 方法将会装箱。根据 C# 基础知识，如果有装箱那就有对象分配

也就是每调用一次如上的方法，将会有一次内存对象的分配

可以通过 EqualityComparer 方法来优化性能，使用 EqualityComparer 可以继续使用泛型判断，可以减少内存分配

```csharp
public override bool Contains(T value)
{
    return EqualityComparer<T>.Default.Equals(_loneEntry, value);
}
```

本文的优化的例子代码请看 [Avoid boxing in FrugalList by stephentoub · Pull Request #4724 · dotnet/wpf](https://github.com/dotnet/wpf/pull/4724 )

更多请看 [【.net 深呼吸】EqualityComparer——自定义相等比较 - 东邪独孤 - 博客园](https://www.cnblogs.com/tcjiaan/p/5700192.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

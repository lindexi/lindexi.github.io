
# dotnet 使用完全对象引用相等判断

默认在列表以及字典哈希这些都会先看对象是否有自己实现的等于判断，如果有就调用对象的。因此对象可以做到两个不同的对象返回相等。而如果需要判断对象引用相等，也就是只有相同的对象才返回相等，此时需要用到 ReferenceEquals 方法。这个判断方法是最快的判断相等的方法，只有在传入的两个参数是相同的对象的时候才会返回 true 的值

<!--more-->


<!-- CreateTime:6/20/2020 11:12:07 AM -->

<!-- 发布 -->

那么如何将这个引用相等放在列表或字典哈希等的判断里面？可以看到字典等的构造函数都有重载方法，要求传入 IEqualityComparer 接口，于是可以做如下实现

```csharp
    class ReferenceEqualsComparer<T> : ReferenceEqualsComparer, IEqualityComparer<T>
    {
        /// <inheritdoc />
        public bool Equals(T x, T y) => ((IEqualityComparer) this).Equals(x, y);

        /// <inheritdoc />
        public int GetHashCode(T obj) => obj.GetHashCode();
    }

    class ReferenceEqualsComparer : IEqualityComparer
    {
        /// <inheritdoc />
        bool IEqualityComparer.Equals(object x, object y) => ReferenceEquals(x, y);

        /// <inheritdoc />
        public int GetHashCode(object obj) => obj?.GetHashCode()??0;
    }
```

对于字典可以这样使用

```csharp
        private Dictionary<Lindexi, string> LindexiShiDoubi { get; } =
            // 这里需要使用完全相等的判断，对象完全相等
            new Dictionary<Lindexi, string>(new ReferenceEqualsComparer<Lindexi>());
```

对于哈希可以这样使用

```csharp
private HashSet<Lindexi> Lindexi { get; } = new HashSet<Lindexi>(new ReferenceEqualsComparer<Lindexi>());
```

这样所有进行判断的 Contains 或 ContainsKey 都使用对象引用判断，只有传入和内存里面存放相同的对象才能判断存在







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
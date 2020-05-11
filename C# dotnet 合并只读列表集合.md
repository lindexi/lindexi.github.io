# C# dotnet 合并只读列表集合

我在写一个对性能要求很高的 WPF 应用，这个应用要求尽可能不要分配内存和尽可能减少GC因为这是一个实时要求特别高的应用。而有功能需要遍历一个由多个不同的数组或列表组成的新列表。本文将告诉大家如何将多个不同的列表或数组合并为一个只读的列表集合，这个只读列表因为只是只读的，可以做到不分配新的列表空间

<!--more-->
<!-- CreateTime:5/10/2020 3:45:49 PM -->


<!-- 不发布 -->

最近在做的一个应用需要运行在特别渣配置的 Lumia 1520 上，而这个 WPF 应用需要有实时性特别强的需求。我准备对他进行很多性能优化，其中就包含了需要合并多个很大的列表为一个传入其他业务里面进行遍历

如果业务使用多个列表合并的新列表只是用来做只读，同时原有的列表也不会存在写入，此时就可以使用本文的方法，自己继承 `IReadOnlyCollection<T>` 或 `IReadOnlyList<T>` 保存原有多个列表的实例作为新的只读列表

```csharp
   public class CombineReadonlyCollection<T> : IReadOnlyCollection<T>
    {
        public CombineReadonlyCollection(params IReadOnlyCollection<T>[] source)
        {
            Source = source;
        }

        public IReadOnlyCollection<T>[] Source { get; }

        public IEnumerator<T> GetEnumerator()
        {
            foreach (var list in Source)
            {
                foreach (var temp in list)
                {
                    yield return temp;
                }
            }
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }

        public int Count => Source.Sum(temp => temp.Count);
    }
```

下面是继承 `IReadOnlyList<T>` 的代码

```csharp
    public class CombineReadonlyList<T> : IReadOnlyList<T>
    {
        public CombineReadonlyList(params IReadOnlyList<T>[] source)
        {
            Source = source;
        }

        public IReadOnlyList<T>[] Source { get; }

        public IEnumerator<T> GetEnumerator()
        {
            foreach (var list in Source)
            {
                foreach (var temp in list)
                {
                    yield return temp;
                }
            }
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }

        public int Count => Source.Sum(temp => temp.Count);

        public T this[int index]
        {
            get
            {
                var n = index;
                var source = Source;

                foreach (var list in source)
                {
                    if (n < list.Count)
                    {
                        return list[n];
                    }

                    n -= list.Count;
                }

                throw new IndexOutOfRangeException();
            }
        }
    }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

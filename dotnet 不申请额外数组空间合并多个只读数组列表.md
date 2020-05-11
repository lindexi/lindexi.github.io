# dotnet 不申请额外数组空间合并多个只读数组列表

我在写一个简单的功能，需要将两个不同的数组合并到一起，但是我的功能只是做只读，如果合并的方法需要申请额外的内存空间，将降低性能。本文写了一个简单的方法，通过判断下标的方法做遍历多个数组组合在一起，通过判断当前获取的下标在对应哪个数组下标范围内，返回对应数组的元素

<!--more-->
<!-- CreateTime:2019/10/9 15:15:10 -->

<!-- csdn -->

合并多个数组或列表有多个不同的方法，但是我找到的方法都需要额外申请内存空间，需要做一次数组元素复制，相对性能比较差，如果是做只读，功能和 Span 相反，那么可以通过遍历的数组下标判断

下面方法可以在项目用，做法很简单，看代码也就知道

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

    public class CombineReadonlyList<T> : IReadOnlyList<T>
    {
        public CombineReadonlyList(params IReadOnlyList<T>[] source)
        {
            Source = source;
        }

        public IReadOnlyList<T>[] Source { get; }

        public IEnumerator<T> GetEnumerator()
        {
            return Source.SelectMany(readOnlyList => readOnlyList).GetEnumerator();
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

这个类如果不算传入的只读列表的原列表的更改，这个类是线程安全的

可能遇到的坑是传入的只读列表的原列表添加了值，也就是 `CombineReadonlyList[n]` 执行两遍获取的元素可能不相同

更多有趣的数组定义请看 [Sakuno.Base.Collections github](https://github.com/KodamaSakuno/Sakuno.Base/tree/master/src/Sakuno.Base/Collections )

如果不需要获取指定下标，那么可以使用 ReadOnlyCollection 请看代码

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
            return Source.SelectMany(readOnlyList => readOnlyList).GetEnumerator();
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }

        public int Count => Source.Sum(temp => temp.Count);
    }
```

如果你觉得 GetEnumerator 方法写的不够快，可以试试下面代码

```csharp
            foreach (var list in Source)
            {
                foreach (var temp in list)
                {
                    yield return temp;
                }
            }
```



本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2472c94382e5f8aaf5f5c689ac78668f1883e8b6/Fujeencemwebaeahale) 欢迎小伙伴访问


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

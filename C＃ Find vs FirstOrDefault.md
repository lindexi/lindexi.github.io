# C＃ Find vs FirstOrDefault

本文告诉大家，在获得数组第一个元素时，使用哪个方法性能更高。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


需要知道，两个方法都是 Linq 的方法，使用之前需要引用 Linq 。对于 List 等都是继承可枚举`Enumerable`这时获取第一个元素可以使用`FirstOrDefault`。如果使用`Find`那么需要数组的类型是`IList`。

下面写一个简单的例子

反编译 Find 可以看到下面代码，下面的代码删了一些代码，让大家比较容易看到 Find 使用的是 for 然后使用判断

```csharp
private T[] _items;

public T Find(Predicate<T> match)
{

  for (int index = 0; index < this._size; ++index)
  {
    if (match(this._items[index]))
      return this._items[index];
  }
  return default (T);
}
```

而 FirstOrDefault 的代码存在 foreach ，这会调用列表的 GetEnumerator 方法，而且还会在结束的时候调用 Dispose 。这样 FirstOrDefault 的性能就比 Find 稍微差一些。

```csharp
public static TSource FirstOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
  foreach (TSource source1 in source)
  {
    if (predicate(source1))
      return source1;
  }
  return default (TSource);
}
```

所以在对于 `List` 类型的获得第一个或默认请使用 Find ，其他的请使用`FirstOrDefault`

 - 对于 List ，使用 for 的速度是 foreach 的两倍

 - 遍历 array 的速度是遍历 List 的两倍

 - 使用 for 遍历 array 的速度是使用 foreach 遍历 List 的5倍

参见：https://stackoverflow.com/a/365658/6116637

[真的要比较 for 和 foreach 的性能吗？（内附性能比较的实测数据） - walterlv](https://walterlv.github.io/post/for-vs-foreach.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
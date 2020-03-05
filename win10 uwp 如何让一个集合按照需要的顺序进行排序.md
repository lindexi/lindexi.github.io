# win10 uwp 如何让一个集合按照需要的顺序进行排序

虽然这是 C# 的技术，但是我是用在 uwp ，于是就把标题写这个名。有一天，我的小伙伴让我优化一个列表。这个列表是 ListView 他绑定了一个 ObservableCollection 所以需要对他做很少的修改。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


我绑定一个数量很多的 ObservableCollection 这个ListView 不能做虚拟化，所以性能问题是如果对这个列表做的修改多，那么速度很慢。我知道的就是一个 数组，他是表示修改后的列表需要的顺序。

于是这个数组我叫 sort ，是一个 `IList<int>` ，他记录了需要排序的顺序

```csharp
IList<int> sort = new List<int>()
{
    0,2,1,3
};
```

上面的代码就是希望把 2 和 1 的位置换一下，当然，实际的是比较复杂的。于是我寻找了两个方法，简单的方法请看下面

```csharp
        private static void Sort<T>(ObservableCollection<T> list, IList<int> sort)
        {
            //按照给出的数组排序
            var source = list.ToList();
            for (int i = 0; i < sort.Count; i++)
            {
                //假如原来的是 0 1 2
                //排序需要 0 2 1
                //那么在sort[1]的时候，发现 list[1] = 1 =2  所以需要在 1 的地方插入 2，同时移除 1 ，这个方法可以获得最少改动
                //如果当前的值和排序要的不相同
                if (list[i].Equals(source[sort[i]]))
                {
                    continue;
                }
                //拿出排序需要的值，插入到现在的位置
                list.Move(sort[i], i);
            }
        }
```

这个方法性能很好，可以使用 ObservableCollection 的 CollectionChanged 查看列表修改的数量

第二个方法是使用插入和删除，因为 Move 只有通知列表才有，为了可以使用 List 的，所以就写了第二个方法。

```csharp
        private static void Sort<T>(IList<T> list, List<int> sort)
        {
            var source = list.ToList();
            for (int i = 0; i < sort.Count; i++)
            {
                if (list[i].Equals(source[sort[i]]))
                {
                    continue;
                }
                var temp = source[sort[i]];
                list.Remove(temp);
                list.Insert(i, temp);
            }
        }
```

可以看到这两个方法的速度都比较好，之前使用的是清除整个列表才添加，可以看到这个方法的速度比上面两个方法差。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792391832.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
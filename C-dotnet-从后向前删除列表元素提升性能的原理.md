
# C# dotnet 从后向前删除列表元素提升性能的原理

如果要从一个列表里面删除一些元素，如何做才能让性能比较高？答案是从列表的后面开始删起，从后到前删除

<!--more-->


<!-- CreateTime:6/13/2020 3:05:27 PM -->



在 dotnet 中的列表存放的底层是一个连续的数组。而列表在删除元素的时候，会通过移动数组的方式让整个列表的元素在内存中依然是连续的

假设我有一个大的列表，此时我删除了第一项，按照上面的说法，列表就需要将后面的所有项移动一次，达到让整个列表的元素在内存是连续

而如果是从后向前开始删除呢？此时列表可能就不需要做移动了，因为从后到前删除，如果刚好后面每一项都需要删除，此时的整个列表无需重新移动元素。而如果不是每一项都需要删除，同时这个列表不关注元素本身的顺序，那么依然还可以优化，优化方法是手动移动元素

假定我是从后向前开始删除元素，这个列表里面的元素不关注元素所在列表的顺序

此时我可以通过将最后一项移动到当前准备删除的元素下标上，然后删除最后一项的方法，让整个列表无需移动元素

一个例子如下：

假设我有列表里面包含元素是 1 2 3 三个元素

此时我从后到前遍历，准备删除元素值是 2 的元素。此时我找到第一个元素 3 不满足，找到第二个元素 2 刚好满足。此时我不是直接删除第二个元素，而是将最后一个元素也就是 3 移动到第二个元素上。然后删除最后一个元素。伪代码如下

```
var lastIndex = 2;
// 将最后一个元素替换掉准备移除的元素
list[1] = list[lastIndex];
list.RemoveAt(lastIndex);
```

因为提前将最后一个元素赋值给准备删除的元素，因此准备删除的元素就没有在列表中被记录，而最后一个元素在列表中被记录了两次。此时删除最后一个元素就可以让最后一个元素在列表中只记录一次，刚好在删除最后一个元素的时候，列表不需要移动元素就能让列表里面所有元素依然是连续在内存存储的

这就是从后向前删除列表元素的原理

在整个 dotnet 的运行时底层有很多这样的逻辑，如[这段代码](https://github.com/dotnet/runtime/blob/1dca350210cbbe3333261d3c823cbbacc3d59758/src/mono/netcore/System.Private.CoreLib/src/System/Threading/TimerQueue.Browser.cs#L65-L87) 就是从后向前开始删除，下面是我简化的代码

```csharp
            for (int i = timers.Count - 1; i >= 0; --i)
            {
                TimerQueue timer = timers[i];
                long waitDurationMs = timer._scheduledDueTimeMs - currentTimeMs;
                if (waitDurationMs <= 0)
                {
                    int lastIndex = timers.Count - 1;
                    if (i != lastIndex)
                    {
                        timers[i] = timers[lastIndex];
                    }
                    timers.RemoveAt(lastIndex);
                }
            }
```

咱使用 [BenchmarkDotNet](https://benchmarkdotnet.org/Guides/ChoosingRunStrategy.htm ) 进行测试对比

先安装 [BenchmarkDotNet](https://benchmarkdotnet.org/Guides/ChoosingRunStrategy.htm ) 库，然后编写代码进行测试，代码比较长，放在本文最后

```xml
    <ItemGroup>
        <PackageReference Include="BenchmarkDotNet" Version="0.12.1"/>
    </ItemGroup>
```

大概的测试效果如下

从 dotnet 的源代码可以看到删除的逻辑大概如下，下面代码有删减

```csharp
        public void RemoveAt(int index)
        {
            _size--;
            if (index < _size)
            {
                Array.Copy(_items, index + 1, _items, index, _size - index);
            }
            else
            {
                _items[_size] = default!;
            }
        }
```

可以看到如果 index 小于 `_size` 就表示删除的不是最后一项，就需要通过 Array.Copy 方法复制一次，让存放的元素依然连续

这是本文的测试代码

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            //var listRemoveTest = new ListRemoveTest();
            //var list = listRemoveTest.GetArgumentList().First();
            //listRemoveTest.RemoveFromEnd(list);

            //if (list.All(temp => temp.N <= ListRemoveTest.MaxCount / 2))
            //{

            //}

            BenchmarkRunner.Run<ListRemoveTest>();
        }
    }

    public class ListRemoveTest
    {
        [Benchmark]
        [ArgumentsSource(nameof(GetArgumentList))]
        public void RemoveAll(List<Foo> list)
        {
            list.RemoveAll(temp => temp.N > MaxCount / 2);
        }

        [Benchmark]
        [ArgumentsSource(nameof(GetArgumentList))]
        public void RemoveFromStart(List<Foo> list)
        {
            for (var i = 0; i < list.Count; i++)
            {
                if (list[i].N > MaxCount / 2)
                {
                    list.RemoveAt(i);
                    i--;
                }
            }
        }

        [Benchmark]
        [ArgumentsSource(nameof(GetArgumentList))]
        public void RemoveFromEnd(List<Foo> list)
        {
            for (var i = list.Count - 1; i >= 0; i--)
            {
                if (list[i].N > MaxCount / 2)
                {
                    var lastIndex = list.Count - 1;
                    if (i != lastIndex)
                    {
                        // 假设列表有值是 1 10 5 而当前 i = 1 而 lastIndex = 2 要移除元素 10 可以先将最后一个值赋值给当前的元素
                        // list[1] = list[lastIndex=2] = 5
                        // 赋值之后的列表是 1 5 5 也就是实际上干掉了 10 这个元素了
                        // 最后再删除多余的最后一个元素就可以了
                        list[i] = list[lastIndex];
                    }

                    list.RemoveAt(lastIndex);
                }
            }
        }

        public const int MaxCount = 20000000;

        public IEnumerable<List<Foo>> GetArgumentList()
        {
            var list = new List<Foo>(MaxCount);
            for (int i = 0; i < MaxCount / 2; i++)
            {
                list.Add(new Foo()
                {
                    N = i
                });
                list.Add(new Foo()
                {
                    N = MaxCount - i
                });
            }

            yield return list;
        }
    }

    public class Foo
    {
        public int N { set; get; }
    }
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
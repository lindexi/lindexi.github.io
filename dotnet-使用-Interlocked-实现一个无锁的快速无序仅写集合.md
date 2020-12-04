
# dotnet 使用 Interlocked 实现一个无锁的快速无序仅写集合

在 dotnet 里面，可以使用 Interlocked 进行原子命令更改 int 等的值，利用这个特性可以在一个固定足够长长度的数组里面，让多线程无锁等待写入值。因为没有锁的存在，无法保证读取时的安全，因此这样的集合只能被设计为只写的集合，只有在业务上完成了所有的写之后，才能作为可读的集合取出来

<!--more-->


<!-- CreateTime:2020/10/12 14:40:23 -->



这是在 [newbe](https://www.newbe.pro/ ) 大佬的代码所看到的用法，这是他的一个实现 [https://github.com/newbe36524/Newbe.Claptrap/blob/a187bac81652f9808a0f6cdc2916bbf6288e8ee3/src/Newbe.Claptrap/Tools/AutoFlushList.cs#](https://github.com/newbe36524/Newbe.Claptrap/blob/a187bac81652f9808a0f6cdc2916bbf6288e8ee3/src/Newbe.Claptrap/Tools/AutoFlushList.cs#) 尽管这个实现里面其实是有很多不安全的

一个安全和推荐的做法是在写入的时候禁止有任何的更改内部数组的长度的行为，同时在写入的时候禁止有任何的读取行为

这个快速无序仅写集合的原理是通过 Interlocked 原子让索引增加，此时每个线程进入写入方法时，都会触发一次索引增加，每次都拿到不同的索引值。而在初始化的时候在集合内容就创建了一个固定长度的数组，这样每次线程进入都会拿到不同的索引值，可以使用索引值对应到数组里面不同的下标，此时进行写入是安全的。当然也是仅写入安全，此时不能做读取

最简的实现方式如下

```csharp
    public class ConcurrentWriteOnlyBag<T>
    {
        public ConcurrentWriteOnlyBag(int capacity)
        {
            Capacity = capacity;
            _buffer = new T[capacity];
        }

        public int Capacity { get; }

        public void Add(T value)
        {
            var currentIndex = Interlocked.Increment(ref _currentIndex);

            if (currentIndex > Capacity)
            {
                throw new ArgumentOutOfRangeException();
            }

            _buffer[currentIndex] = value;
        }

        private readonly T[] _buffer;

        private int _currentIndex = -1;
    }
```

可以看到上面代码只有写入的功能，即使在写入完成之后，也没有方法去读取内部的 `_buffer` 数组的内容，因此可以在上面的类加上下面方法

```csharp
        /// <summary>
        /// 非线程安全
        /// </summary>
        /// <returns></returns>
        public IReadOnlyCollection<T> GetReadOnlyCollection() => _buffer;
```

需要注意的是 `GetReadOnlyCollection` 方法仅能在这个 ConcurrentWriteOnlyBag 写入完全完成之后才能使用，需要业务端保证这个行为，否则会出现读取的数据是不安全的数据，例如写入了一半或没有写入

上面代码的 GetReadOnlyCollection 方法是将整个内部 `_buffer` 全部返回，而不是将当前已写入的索引返回，因此在实际使用的时候，如果不会完全写满，还需要大家自己加上这部分的功能

因为这个集合没有任何的锁的存在，在多线程同时写入的时候的性能超级好

我有写了一些测试的代码，本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/3efbbbdd/JeekoheabeNurnurdawjerewear ) 欢迎小伙伴访问

另外推荐一下 [newbe36524/Newbe.Claptrap](https://github.com/newbe36524/Newbe.Claptrap ) 这个有趣的库

同时推荐大家多线程工具集合：[https://github.com/dotnet-campus/AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection) 开源库

在 [https://github.com/dotnet-campus/AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection) 包含了 AsyncQueue DoubleBuffer DoubleBufferTask AsyncAutoResetEvent AsyncManualResetEvent LimitedRunningCountTask ExecuteOnceAwaiter 等的实现，详细请看

- [dotnet 使用 AsyncQueue 创建高性能内存生产者消费者队列](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-AsyncQueue-%E5%88%9B%E5%BB%BA%E9%AB%98%E6%80%A7%E8%83%BD%E5%86%85%E5%AD%98%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85%E9%98%9F%E5%88%97.html )
- [dotnet 双缓存数据结构设计 下载库的文件写入缓存框架](https://blog.lindexi.com/post/dotnet-%E5%8F%8C%E7%BC%93%E5%AD%98%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E8%AE%BE%E8%AE%A1-%E4%B8%8B%E8%BD%BD%E5%BA%93%E7%9A%84%E6%96%87%E4%BB%B6%E5%86%99%E5%85%A5%E7%BC%93%E5%AD%98%E6%A1%86%E6%9E%B6.html )
- [C# dotnet 高性能多线程工具 AsyncAutoResetEvent 异步等待使用方法和原理](https://blog.lindexi.com/post/C-dotnet-%E9%AB%98%E6%80%A7%E8%83%BD%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%B7%A5%E5%85%B7-AsyncAutoResetEvent-%E5%BC%82%E6%AD%A5%E7%AD%89%E5%BE%85%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%E5%92%8C%E5%8E%9F%E7%90%86.html )
- [C# dotnet 高性能多线程工具 ExecuteOnceAwaiter 只执行一次的任务](https://lindexi.gitee.io/post/C-dotnet-%E9%AB%98%E6%80%A7%E8%83%BD%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%B7%A5%E5%85%B7-ExecuteOnceAwaiter-%E5%8F%AA%E6%89%A7%E8%A1%8C%E4%B8%80%E6%AC%A1%E7%9A%84%E4%BB%BB%E5%8A%A1.html )







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# dotnet 双缓存数据结构设计 下载库的文件写入缓存框架

我在写一个文件下载库，这个下载库利用断点续传机制，支持多线程下载一个文件。但是文件写入只能支持单线程，我不想让网络下载需要等待磁盘写入，因此我需要先在内存做缓存，然后让磁盘写入。配合 DirectX 渲染的设计方法，采用双缓存数据结构设计，也就是有两个集合，其中一个集合用来被其他模块写入，另一个集合用来作为当前使用。此时能做到网络下载使用的集合和文件写入的集合不是相同的一个集合，因此两部分的速度差异将不会相互影响

<!--more-->
<!-- CreateTime:2020/9/11 8:45:25 -->

<!-- 发布 -->

这个文件下载库在 GitHub 完全开源，欢迎小伙伴点击 Star 和参与开发

[dotnet-campus/dotnetCampus.FileDownloader: The repo includes the file download library and the file download tool.](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/ )

可以进行试用的方法是通过 dotnet tool 工具使用，命令行版本使用方法如下

```
dotnet tool install -g dotnetCampus.FileDownloader.Tool
DownloadFile -u [the download url] -o [the download file]
```

下载方法是在命令输入 `DownloadFile -u [下载链接] -o [下载文件路径]` 

带 GUI 的 WPF 版本请使用下面命令安装

```csharp
dotnet tool update -g dotnetCampus.FileDownloader.WPF
```

在控制台输入 `dotnetCampus.FileDownloader.WPF` 即可启动简单版本的例子

而更多的是这个下载库是通过 NuGet 库的方式，可以让你在其他项目里面引用这个库


可以使用下面代码给项目添加下载库的引用

```csharp
dotnet add package dotnetCampus.FileDownloader
```

使用方法如下

```csharp
var segmentFileDownloader = new SegmentFileDownloader(url, file);

await segmentFileDownloader.DownloadFileAsync();
```

好的，广告就到这里

我在写下载库遇到的问题是网络下载速度和磁盘写入速度有差异，我不期望网络下载需要等待磁盘下载，因此我抄袭了 DirectX 的设计方法，开了一个双缓存。刚好这个文件写入双缓存类足够通用，可以让我水一篇博客

如果只是想要抄代码的小伙伴，请到文本最后面

这个双缓存类的设计里面需要有两个集合，一个集合用于被加入，另一个集合用于被使用。因此想要这个类足够通用，就需要让集合使用 ICollection 让上层可以注入

而因此具体放的元素是不需要关注的，因此可以作为上层注入，也就是这个类有两个泛形

```csharp
    /// <summary>
    /// 提供双缓存 线程安全列表
    /// </summary>
    /// 写入的时候写入到一个列表，通过 SwitchBuffer 方法，可以切换当前缓存
    class DoubleBuffer<T, TU> where T : class, ICollection<TU>
    {

    }
```

接下来定义两个缓存

```csharp
        private T AList { get; }
        private T BList { get; }
```

还需要再定义一个当前使用的被写入的缓存属性，这个属性将会指向 AList 或 BList 其中一个值

```csharp
        private T CurrentList { set; get; }
```

在初始化的时候将会初始化这三个属性

```csharp
        public DoubleBuffer(T aList, T bList)
        {
            AList = aList;
            BList = bList;

            CurrentList = AList;
        }
```

定义加入元素的方法，加入的时候和缓存切换的时候都需要加上锁，因此还需要定义一个对象

```csharp
        private readonly object _lock = new object();

        public void Add(TU t)
        {
            lock (_lock)
            {
                CurrentList.Add(t);
            }
        }
```

再定义一个切换缓存的方法

```csharp
        public T SwitchBuffer()
        {
            lock (_lock)
            {
                if (ReferenceEquals(CurrentList, AList))
                {
                    CurrentList = BList;
                    return AList;
                }
                else
                {
                    CurrentList = AList;
                    return BList;
                }
            }
        }
```

可以看到上面的缓存切换是十分快的，但是这里存在一个坑，也就是返回的 T 不能被保存，只能用一次，同时也禁止多线程同时调用

上面代码的切换缓存方法只能使用一个线程调用，同步调用。调用的时候返回的集合不能被保存，也就是如下代码是不推荐的

```csharp
private void Foo()
{
   Buffer = fooDoubleBuffer.SwitchBuffer();
}

private List<int> Buffer { set; get; }
```

而且有需求是执行完成当前缓存里面的所有任务，而在执行任务的过程中有其他线程加入新的任务，因此就封装了一个方法，调用这个方法传入执行任务的委托就可以实现

```csharp
        /// <summary>
        /// 执行完所有任务
        /// </summary>
        /// <param name="action">当前缓存里面存在的任务，请不要保存传入的 List 参数</param>
        public void DoAll(Action<T> action)
        {
            while (true)
            {
                var buffer = SwitchBuffer();
                if (buffer.Count == 0) break;

                action(buffer);
                buffer.Clear();
            }
        }
```

使用上面这个方法大概能执行完成这个双缓存里面的所有任务，但是上面这个是同步的方法，于是再添加一个支持异步的方法

```csharp
        /// <summary>
        /// 执行完所有任务
        /// </summary>
        /// <param name="action">当前缓存里面存在的任务，请不要保存传入的 List 参数</param>
        /// <returns></returns>
        public async Task DoAllAsync(Func<T, Task> action)
        {
            while (true)
            {
                var buffer = SwitchBuffer();
                if (buffer.Count == 0) break;

                await action(buffer);
                buffer.Clear();
            }
        }
```

不过这个双缓存也不能完成在不断有其他线程加入任务的时候，执行完成所有。因为在返回之前的 `if (buffer.Count == 0)` 判断的时候，也许此时又有其他线程加入了任务。但是作为文件写入的双缓存是可以在网络下载完成之后，再次调用 DoAllAsync 方法，只要在 DoAllAsync 方法调用之前就不会存在有新任务加入，那么这个方法是可以完全执行完成所有任务

但是如果需要手动写执行完成所有的调用方法，那么这部分代码也许会写出线程相关的逻辑，因此再封装一个 DoubleBufferTask 类，这是一个使用双缓存的任务调度类

这个类可以支持设置任意的类型作为任务的数据，同时传入处理任务的执行方法

```csharp
    class DoubleBufferTask<T>
    {
        public DoubleBufferTask(Func<List<T>, Task> doTask)
        {
            _doTask = doTask;
        }

        private readonly Func<List<T>, Task> _doTask;
    }
```

对 DoubleBufferTask 的使用方法就是创建起来，然后传入执行任务的方法，接着可以多线程调用 AddTask 方法添加任务，在所有任务加入完成之后，调用 Finish 方法表示完成

```csharp
        public void AddTask(T t)
        {
            DoubleBuffer.Add(t);
            
            // 忽略代码
        }

        public void Finish()
        {
            // 忽略代码
        }

        private DoubleBuffer<T> DoubleBuffer { get; } = new DoubleBuffer<T>();
```

同时还需要有一个 WaitAllTaskFinish 方法给上层，可以用来等待调用 Finish 方法之后所有任务执行完成

```csharp
        public Task WaitAllTaskFinish()
        {
            return FinishTask.Task;
        }

        private TaskCompletionSource<bool> FinishTask { get; } = new TaskCompletionSource<bool>();
```

使用 FinishTask 的优势在于可以在调用 Finish 方法之后，调用 WaitAllTaskFinish 也能返回。有多个线程同时等待 WaitAllTaskFinish 方法也能线程安全返回

本文的全部代码

```csharp

    /// <summary>
    /// 提供双缓存 线程安全列表
    /// </summary>
    /// 写入的时候写入到一个列表，通过 SwitchBuffer 方法，可以切换当前缓存
    class DoubleBuffer<T> : DoubleBuffer<List<T>, T>
    {
        public DoubleBuffer() : base(new List<T>(), new List<T>())
        {
        }
    }

    /// <summary>
    /// 提供双缓存 线程安全列表
    /// </summary>
    /// 写入的时候写入到一个列表，通过 SwitchBuffer 方法，可以切换当前缓存
    class DoubleBuffer<T, TU> where T : class, ICollection<TU>
    {
        public DoubleBuffer(T aList, T bList)
        {
            AList = aList;
            BList = bList;

            CurrentList = AList;
        }

        public void Add(TU t)
        {
            lock (_lock)
            {
                CurrentList.Add(t);
            }
        }

        public T SwitchBuffer()
        {
            lock (_lock)
            {
                if (ReferenceEquals(CurrentList, AList))
                {
                    CurrentList = BList;
                    return AList;
                }
                else
                {
                    CurrentList = AList;
                    return BList;
                }
            }
        }

        /// <summary>
        /// 执行完所有任务
        /// </summary>
        /// <param name="action">当前缓存里面存在的任务，请不要保存传入的 List 参数</param>
        public void DoAll(Action<T> action)
        {
            while (true)
            {
                var buffer = SwitchBuffer();
                if (buffer.Count == 0) break;

                action(buffer);
                buffer.Clear();
            }
        }

        /// <summary>
        /// 执行完所有任务
        /// </summary>
        /// <param name="action">当前缓存里面存在的任务，请不要保存传入的 List 参数</param>
        /// <returns></returns>
        public async Task DoAllAsync(Func<T, Task> action)
        {
            while (true)
            {
                var buffer = SwitchBuffer();
                if (buffer.Count == 0) break;

                await action(buffer);
                buffer.Clear();
            }
        }

        private readonly object _lock = new object();

        private T CurrentList { set; get; }

        private T AList { get; }
        private T BList { get; }
    }

    class DoubleBufferTask<T>
    {
        public DoubleBufferTask(Func<List<T>, Task> doTask)
        {
            _doTask = doTask;
        }

        public void AddTask(T t)
        {
            DoubleBuffer.Add(t);

            DoInner();
        }

        private async void DoInner()
        {
            // ReSharper disable once InconsistentlySynchronizedField
            if (_isDoing) return;

            lock (DoubleBuffer)
            {
                if (_isDoing) return;
                _isDoing = true;
            }

            await DoubleBuffer.DoAllAsync(_doTask);

            lock (DoubleBuffer)
            {
                _isDoing = false;
                Finished?.Invoke(this, EventArgs.Empty);
            }
        }

        public void Finish()
        {
            lock (DoubleBuffer)
            {
                if (!_isDoing)
                {
                    FinishTask.SetResult(true);
                    return;
                }

                Finished += (sender, args) => FinishTask.SetResult(true);
            }
        }

        public Task WaitAllTaskFinish()
        {
            return FinishTask.Task;
        }

        private TaskCompletionSource<bool> FinishTask { get; } = new TaskCompletionSource<bool>();

        private bool _isDoing;

        private event EventHandler? Finished;

        private readonly Func<List<T>, Task> _doTask;

        private DoubleBuffer<T> DoubleBuffer { get; } = new DoubleBuffer<T>();
    }
```

本文的双缓存库在 GitHub 独立发布，请看 [https://github.com/dotnet-campus/AsyncWorkerCollection/](https://github.com/dotnet-campus/AsyncWorkerCollection/)  可以在 NuGet 上安装

本文的 [AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection/) 库提供两个 NuGet 包，一个是 dll 引用，另一个是源代码引用，分别如下

- dotnetCampus.AsyncWorkerCollection
- dotnetCampus.AsyncWorkerCollection.Source

如果使用 SDK 版本的 csproj 可以在项目文件使用下面代码安装源代码版本

```xml
    <ItemGroup>
        <PackageReference Include="dotnetCampus.AsyncWorkerCollection.Source" Version="1.2.1">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
    </ItemGroup>
```

安装完成之后即可使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

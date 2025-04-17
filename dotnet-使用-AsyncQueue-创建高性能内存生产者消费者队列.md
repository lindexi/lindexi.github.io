
# dotnet 使用 AsyncQueue 创建高性能内存生产者消费者队列

在本机相同进程中创建生产者消费者队列，可以解决很多线程安全以及高性能需求问题。本文告诉大家如何通过在 GitHub 完全开源的 AsyncWorkerCollection 库的 AsyncQueue 类创建在内存中的高性能低资源占用的生产者消费者队列

<!--more-->


<!-- CreateTime:2020/9/21 8:43:42 -->




本文使用的 AsyncWorkerCollection 库在 GitHub 完全开源，请看 [https://github.com/dotnet-campus/AsyncWorkerCollection/](https://github.com/dotnet-campus/AsyncWorkerCollection/) 

这个库里面的所有代码都是在我团队实际项目经过约3年的测试，在大约 200 万台设备上运行过

本文使用的 AsyncQueue 是 [AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection/) 库的其中一个类。通过 AsyncQueue 可以制作出一个支持多线程并发调用的生产者消费者队列，而 AsyncQueue 本身支持异步，可以配置线程池做到极低的资源占用

## 意义

为什么需要有生产者消费者队列？其实这个设计方法可以极大规避多个模块之间的性能差异，如下面例子

我有两个不同的模块，假定是 A 和 B 两个。业务逻辑要求让 A 模块执行完成的数据，进入到 B 模块。换句话说就是 B 模块的处理都需要依赖 A 模块的执行完成

但是现在存在的问题是 A 和 B 两个模块的执行速度有差异。如 A 模块是通过读取本机文件，而 B 模块是解析文件本身。或者说 A 模块是接收网络请求，而 B 模块是执行复杂的数据库逻辑

那么就会存在一个问题，能否让 A 和 B 独立执行，同时控制 A 和 B 两个模块的线程数量，以达到整体性能最佳？此时通过生产者消费者队列就能实现

按照如上描述，其实 A 模块就是生产者，生产出数据让 B 模块处理。而 B 模块就是消费者

如果 A 和 B 两个模块设计为独立执行，那么意味着可以让 A 和 B 两个模块的执行线程数量可以不匹配。如 A 的速度比较慢，此时分配给 A 更多的线程，或者说 B 执行比较慢分配给 B 更多的线程处理

在使用生产者消费者队列另一个意义在于可以做到资源的动态调配。如我有一个不重要的模块，这个模块需要处理一些杂务，而我不期望给这个模块投入太大的资源，但是我又期望在我应用空闲的时候可以将空闲资源投入处理。此时使用 生产者消费者队列 就能完成这个需求，所有模块将任务投入到生产者消费者队列里面，而平时只有很少的线程投入使用作为消费者处理这些任务。在应用空闲的时候，将更多线程投入到消费者处理里面处理

当然生产者消费者队列可以使用的业务将会很多，其他用途还请小伙伴自己摸索，或者百度一下

大部分的生产者消费者队列库都是设计为分布式的，支持多设备跨进程的，而这些库也就需要使用更多的资源。本文的 AsyncQueue 是在内存中创建，不会涉及到数据库等功能，只能在相同进程内使用。而因为没有跨进程和设备的功能，可以减少很多资源的时候，只需要一个简单的信号量锁就能完成

## 安装库

在使用之前的第一步就是安装 NuGet 库，本文的 [AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection/) 库提供两个 NuGet 包，一个是 dll 引用，另一个是源代码引用，分别如下

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

## 使用方法

本质上的 AsyncQueue 的设计就是一个队列，因此用法和队列相同，有一个叫入队的方法，还有一个叫出队的方法。入队的方法是给生产者使用的，将数据或任务加入到队列里面提供给消费者使用。而出队就是给消费者使用的，消费者通过出队获取数据或任务用来执行

假定有数据是 FooTask 类，在创建 AsyncQueue 对象就需要传入对应的数据或任务的定义

```csharp
    var asyncQueue = new AsyncQueue<FooTask>();
```

在生产者对应的代码里面使用 Enqueue 方法入队，这个方法是线程安全的，可以随意调用

```csharp
   asyncQueue.Enqueue(new FooTask());
```

在消费者对应的代码里面使用 DequeueAsync 方法出队，这个方法是线程安全的，在队列里面没有数据的时候将会通过 await 等待，让线程返回线程池。在有数据的时候此方法将会返回。每一条数据都只会返回一次，也就是如果有多个线程同时调用 DequeueAsync 方法不会存在返回同一条数据。换句话说有多少次 Enqueue 方法入队，就会有多少次 DequeueAsync 的返回

```csharp
    var fooTask = await asyncQueue.DequeueAsync();
```

使用方法就这么简单

那么如何用来做修改执行的线程的数量？如上面说的，本文的 AsyncQueue 是线程安全的，支持多个线程调用。因此如果修改调用 Enqueue 入队的线程数量就能修改生产者的线程数量。而修改等待 DequeueAsync 返回的线程数量也就能修改消费者的线程数量

也可以用来固定消费者的线程数量，用法很简单，就是预设置对应的消费者线程的数量就可以。如设置有三个线程进入循环等待 DequeueAsync 返回，那么消费者将最多使用三个线程执行。而在等待的时候通过 await 可以让线程返回线程池提升性能

有一点需要注意的是 AsyncQueue 是包含 Dispose 和 DisposeAsync 方法的，因为本质 AsyncQueue 使用了锁，需要手动释放。但主要需要调用的原因是让在业务完成之后，让没有手动释放的 DequeueAsync 方法可以释放，解决内存泄露问题。这部分原理请看 [dotnet 使用 SemaphoreSlim 可能的内存泄露](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SemaphoreSlim-%E5%8F%AF%E8%83%BD%E7%9A%84%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2.html)  大概的问题是在 AsyncQueue 对象不再使用的时候，如果有业务代码在 DequeueAsync 等待，那么这些业务代码引用的类将会存在内存泄露，不会被释放

在调用 Dispose 或 DisposeAsync 方法将可以释放 AsyncQueue 对象，在执行完成当前队列里面所有的数据之后，最后的 DequeueAsync 返回，解决内存泄露。而 Dispose 和 DisposeAsync 方法的不同在于，调用 DisposeAsync 方法相当于调用如下代码

```csharp
    asyncQueue.Dispose();
    await asyncQueue.WaitForCurrentFinished();
```

## 例子

下面用一个例子让小伙伴了解使用的方法，和这个库的强大

假定有任务的定义如下

```csharp
    class FooTask
    {
        public void Do()
        {
            Console.WriteLine("DoTask");
        }
    }
```

在实际使用的时候，其实更多将会定义为数据本身，让消费者执行相同的逻辑处理数据。而定义为任务本身可以提升灵活度，也就是每个任务可以使用不同的逻辑，但是需要小心任务本身的线程安全问题。如在 WPF 中不应该使用非 UI 线程访问 UI 线程控件等

这个任务有 100 个从线程池拿到的线程在创建，加入队列


```csharp
            var random = new Random();
            var asyncQueue = new AsyncQueue<FooTask>();

            for (int i = 0; i < 100; i++)
            {
                Task.Run(async () =>
                {
                    while (true)
                    {
                        asyncQueue.Enqueue(new FooTask());

                        await Task.Delay(random.Next(1000));
                    }
                });
            }
```

而有对应的 10 个从线程池拿到的线程在执行

```csharp
            for (int i = 0; i < 10; i++)
            {
                Task.Run(async () =>
                {
                    while (true)
                    {
                        var fooTask = await asyncQueue.DequeueAsync();
                        fooTask.Do();
                        Console.WriteLine($"剩余 {asyncQueue.Count}");
                        await Task.Delay(random.Next(50));
                    }
                });
            }
```

执行代码可以看到在消费线程里面将会不断从队列里面拿到任务，然后调用任务


本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/6773a38647b3b2e288c3383e252c3501b4e6ec0c/YearnakeenallKawlawberekubayqe)欢迎小伙伴访问

而 AsyncQueue 不仅可以用在多线程并发调用，也可以支持在单线程玩出协程的方法。先调用 DequeueAsync 加上等待，此时将会在当前线程注册等待调用，接着在其他业务模块调用入队的方法，每次调用入队将会回到出队的异步方法里面

如在 WPF 中添加下面代码

```csharp
        private static async void DoTask(AsyncQueue<FooTask> asyncQueue)
        {
            while (true)
            {
                var task = await asyncQueue.DequeueAsync();
                task.Do();
                Console.WriteLine($"执行线程 {Thread.CurrentThread.ManagedThreadId}");
            }
        }

            var random = new Random();
            var asyncQueue = new AsyncQueue<FooTask>();

            DoTask(asyncQueue);

            for (int i = 0; i < 100; i++)
            {
                asyncQueue.Enqueue(new FooTask());

                await Task.Delay(random.Next(1000));
            }
```

可以看到里面代码全程没有创建线程也没有调用线程池，在入队之后将会在入队的线程调用到 await 出让才会让 DoTask 继续执行

如果代码不是在 WPF 中使用，而是在控制台就需要自己实现同步上下文，请看 [C# dotnet 自己实现一个线程同步上下文](https://blog.lindexi.com/post/C-dotnet-%E8%87%AA%E5%B7%B1%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E4%B8%8A%E4%B8%8B%E6%96%87.html)

## 原理

其实这个 AsyncQueue 的本质就是使用 ConcurrentQueue 和 SemaphoreSlim 两个基础类创建的

关于 SemaphoreSlim 请看 [C# dotnet 的锁 SemaphoreSlim 和队列](https://blog.lindexi.com/post/C-dotnet-%E7%9A%84%E9%94%81-SemaphoreSlim-%E5%92%8C%E9%98%9F%E5%88%97.html)

这个 SemaphoreSlim 锁的功能就是提供信号量，和异步等待的功能。信号量的用法就是设置多少次信号量就允许多少次使用信号量，这就是 AsyncQueue 可以让入队和出队的最大次数相等的原因

为什么是说最大次数而不是次数？原因是在于可以入队，但是没有线程调用 DequeueAsync 出队

在 DequeueAsync 方法底层调用的等待就是调用 SemaphoreSlim 的等待方法，如果没有信号量可以使用，那么这个等待将会等待到有信号量被设置。而等待是异步方法，也就是不会占用一个线程，此时占用的资源很小。当然用这个方法就需要小心 [dotnet 使用 SemaphoreSlim 可能的内存泄露](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SemaphoreSlim-%E5%8F%AF%E8%83%BD%E7%9A%84%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2.html) 这也就是在不使用 AsyncQueue 需要调用释放的原因

而 ConcurrentQueue 就是提供本身存放数据的类，这个类的设计就是线程安全的

因此通过 ConcurrentQueue 存放数据，而通过 SemaphoreSlim 通知出队，让 SemaphoreSlim 支持等待数据出队和让入队数量和出队最大数量相等

使用这两个类的配合就可以做到 AsyncQueue 的高性能低资源占用

本文使用的 [AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection/) 库是完全开源基于 MIT 协议的库，欢迎小伙伴使用，在使用中遇到任何问题都欢迎在 GitHub 讨论

更多请看

- [dotnet 使用 SemaphoreSlim 可能的内存泄露](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SemaphoreSlim-%E5%8F%AF%E8%83%BD%E7%9A%84%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2.html) 
- [C# dotnet 的锁 SemaphoreSlim 和队列](https://blog.lindexi.com/post/C-dotnet-%E7%9A%84%E9%94%81-SemaphoreSlim-%E5%92%8C%E9%98%9F%E5%88%97.html)
- [C# dotnet 自己实现一个线程同步上下文](https://blog.lindexi.com/post/C-dotnet-%E8%87%AA%E5%B7%B1%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E4%B8%8A%E4%B8%8B%E6%96%87.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
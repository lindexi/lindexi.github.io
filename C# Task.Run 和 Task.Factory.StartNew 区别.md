# C# Task.Run 和 Task.Factory.StartNew 区别

有小伙伴问我，为什么不推荐他使用 Task.Factory.StartNew ，因为 Task.Run 是比较新的方法。

本文告诉大家 Task.Run 和 Task.Factory.StartNew 区别

<!--more-->
<!-- CreateTime:2019/1/29 16:14:52 -->


有很多博客说到了 Task.Run 和 Task.Factory.StartNew 区别，所以我也就不需要展开告诉大家。

只需要知道 `Task.Run` 是在 dotnet framework 4.5 之后才可以使用，但是 `Task.Factory.StartNew` 可以使用比 `Task.Run` 更多的参数，可以做到更多的定制。

可以认为 `Task.Run` 是简化的 `Task.Factory.StartNew` 的使用，除了需要指定一个线程是长时间占用的，否则就使用 `Task.Run` 

## 创建新线程

下面来告诉大家使用两个函数创建新的线程

```csharp
            Task.Run(() =>
            {
                var foo = 2;
            });
```

这时 foo 的创建就在另一个线程，需要知道 Task.Run 用的是线程池，也就是不是调用这个函数就会一定创建一个新的线程，但是会在另一个线程运行。

```csharp
            Task.Factory.StartNew(() =>
            {
                var foo = 2;
            });
```

可以看到，两个方法实际上是没有差别，但是`Task.Run`比较好看，所以推荐使用`Task.Run`。

## 等待线程

创建的线程，如果需要等待线程执行完成在继续，那么可以使用 await 等待

```csharp
        private static async void SeenereKousa()
        {
            Console.WriteLine("开始 线程"+Thread.CurrentThread.ManagedThreadId);
            await Task.Run(() =>
            {
                Console.WriteLine("进入 线程" + Thread.CurrentThread.ManagedThreadId);
            });
            Console.WriteLine("退出 线程"+Thread.CurrentThread.ManagedThreadId);
        }
```

但是需要说的是这里使用 await 主要是给函数调用的外面使用，上面代码在函数里面使用 await 函数是 void 那么和把代码放在 task 里面是相同

```csharp
        private static async void SeenereKousa()
        {
            Console.WriteLine("开始 线程"+Thread.CurrentThread.ManagedThreadId);
            await Task.Run(() =>
            {
                Console.WriteLine("进入 线程" + Thread.CurrentThread.ManagedThreadId);
                Console.WriteLine("退出 线程"+Thread.CurrentThread.ManagedThreadId);
            });
        }
```

但是如果把 void 修改为 Task ，那么等待线程才有用

除了使用 await 等待，还可以使用 WaitAll 等待

```csharp
            Console.WriteLine("开始 线程" + Thread.CurrentThread.ManagedThreadId);
            var t = Task.Run(() =>
            {
                Console.WriteLine("进入 线程" + Thread.CurrentThread.ManagedThreadId);
            });

            Task.WaitAll(t);

            Console.WriteLine("退出 线程" + Thread.CurrentThread.ManagedThreadId);
```

使用 WaitAll 是在调用 WaitAll 的线程等待，也就是先在线程 1 运行，然后异步到 线程2 运行，这时线程1 等待线程2运行完成再继续，所以输出

```csharp
开始 线程1
进入 线程2
退出 线程1
```

## 长时间运行

两个函数最大的不同在于 `Task.Factory.StartNew ` 可以设置线程是长时间运行，这时线程池就不会等待这个线程回收

```csharp
            Task.Factory.StartNew(() =>
            {
                for (int i = 0; i < 100; i++)
                {
                    var foo = 2;
                }
                Console.WriteLine("进行 线程" + Thread.CurrentThread.ManagedThreadId);
            }, TaskCreationOptions.LongRunning);
```

所以在需要设置线程是长时间运行的才需要使用 `Task.Factory.StartNew ` 不然就使用 `Task.Run`

调用 `Task.Run(foo)` 就和使用下面代码一样

```csharp
Task.Factory.StartNew(foo, 
    CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
```

实际上 `Task.Run(foo)` 可以认为是对 `Task.Factory.StartNew` 封装，使用简单的默认的参数。如果需要自己定义很多参数，就请使用 `Task.Factory.StartNew` 定义参数。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

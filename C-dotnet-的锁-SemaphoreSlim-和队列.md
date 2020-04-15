
# C# dotnet 的锁 SemaphoreSlim 和队列

本文主要是试验在顺序进入等待 SemaphoreSlim 的任务是否会按照顺序经过锁执行

<!--more-->


<!-- 发布 -->

我在一个有趣的WPF程序里面，需要限制任务同时执行的线程数量，不然用户就会说用我的程序会让电脑卡渣。而我的任务是需要按照指定顺序执行的，我需要每次同时仅执行10个任务，同时任务执行按照传入的顺序

此时可以用到 SemaphoreSlim 这个类，这个类的作用如下，给定初始的可以通过锁的数量，以及这个最大可以通过锁的数量。通过 Wait 方法进行等待，如果当前已经有超过可以通过的任务通过了，那么在 Wait 方法将会阻塞。如果没有超过可以通过的数量，那么将可以通过

使用 Release 方法可以添加一个或多个可以通过的数量，但是可以通过的数量最大不会超过初始化时传入的最大可以通过锁的数量的值

如下面代码

```csharp
            var semaphoreSlim = new SemaphoreSlim(10, 20);
```

此时表示初始化的时候，可以让 10 个任务通过锁，也就是初始化的时候可以有10次调用 Wait 方法能通过

而第二个参数表示最大的可以通过的数量，通过 Release 可以添加一个或多个可以通过锁的任务，如 `semaphoreSlim.Release(100);` 表示设置有 100 个可以通过锁的任务，但是实际上在上面代码里面，因为设置了最大值是 20 也就是即使写 100 其实之后能通过的任务只有 20 个

而本文的测试在于我有任务按照顺序调用 Wait 方法进入等待，如我的任务有序号，我按照任务1 任务2 任务3 的顺序调用Wait方法，同时此时的锁的可以通过的数量是 0 也就是所有任务在等待

之后我通过 Release 方法的不断调用，请问此时通过锁的任务是否和队列一样，先等待的任务就先通过锁。答案是这样的

先调用 Wait 方法的任务，在锁开始释放的时候就先通过，我通过一个有趣的代码用来测试

我需要有很多线程进入锁的 Wait 方法，但是这些线程每个线程是一个任务，这些任务有顺序，进入等待方法的时候按照顺序进入

而小伙伴都知道，创建线程的先后顺序不会等于线程执行的先后顺序，所以我使用了 AutoResetEvent 在线程创建然后执行开始之后再创建下一个线程

先通过 SemaphoreSlim 创建一个初始值是 10 而最大值是 10 的锁，然后创建一个 AutoResetEvent 设置默认能通过一次

```csharp
            var semaphoreSlim = new SemaphoreSlim(10, 10);

        private static readonly AutoResetEvent _autoResetEvent = new AutoResetEvent(true);
```

接下来进入循环创建线程，创建线程的时候先等待 AutoResetEvent 锁，而在线程执行的时候释放 AutoResetEvent 锁，这样就能让线程一定是在上一个线程执行之后再创建。而设置 AutoResetEvent 的初始值是通过，也就是第一个线程可以创建，但第二个线程需要等待第一个线程开始执行再创建

```csharp
            for (int i = 0; i < 1000; i++)
            {
                var n = i;
                _autoResetEvent.WaitOne();
                new Thread(() => { GeregelkunoNeawhikarcee(semaphoreSlim, n); }).Start();
            }
```

添加 GeregelkunoNeawhikarcee 方法，在方法进入的时候，也就是线程开始执行，释放 AutoResetEvent 锁，这样就能让下一个线程创建

```csharp
            _autoResetEvent.Set();
```

进入等待 SemaphoreSlim 此时等待是按照线程创建的顺序等待

```csharp
            semaphoreSlim.Wait();
```

接下来输出当前的任务号，主要用来调试是否通过锁的顺序和线程进入等待的顺序相同

```csharp
            Console.WriteLine(n);
```

接下来通过 Thread.Sleep 模拟执行长任务

在任务执行完成之后释放锁让下一个任务开始，全部代码放在这里

```csharp
        static void Main(string[] args)
        {
            var semaphoreSlim = new SemaphoreSlim(10, 10);

            for (int i = 0; i < 1000; i++)
            {
                var n = i;
                _autoResetEvent.WaitOne();
                new Thread(() => { GeregelkunoNeawhikarcee(semaphoreSlim, n); }).Start();
            }

            Console.Read();
        }

        private static readonly AutoResetEvent _autoResetEvent = new AutoResetEvent(true);

        private static void GeregelkunoNeawhikarcee(SemaphoreSlim semaphoreSlim, int n)
        {
            Console.WriteLine($"{n} 进入");
            _autoResetEvent.Set();

            semaphoreSlim.Wait();
            Console.WriteLine(n);

            Thread.Sleep(TimeSpan.FromSeconds(1));
            semaphoreSlim.Release();
        }
```

可以看到代码是按照顺序输出的

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/cf8c1add01a571bafeb0548b6aa43da8670227c9/CallnernawbawceKairwemwhejeene)欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
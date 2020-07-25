
# dotnet 使用 SemaphoreSlim 可能的内存泄露

在使用 SemaphoreSlim 这个锁，能做到的是指定让任务执行几次，同时提供异步方法，减少线程占用。但异步的方法如果没有用对，会因为异步状态机的引用，而存在内存泄露

<!--more-->


<!-- 发布 -->

在 dotnet 的 SemaphoreSlim 的用法基本上是一个线程调用 WaitAsync 等待其他线程调用 Release 释放，在 Release 方法可以设置释放几次，设置之后就能通过几次的 WaitAsync 方法

调用 WaitAsync 方法，如果使用 `await` 那么将会出让线程执行权，意思是如果是线程池的线程，可以让线程回到线程池，让这个线程去执行其他任务

因此使用 SemaphoreSlim 的 WaitAsync 方法总体性能比较好

但是如果在调用 WaitAsync 方法之后，其他线程调用了 Release 的代码，那么如何让线程从 WaitAsync 方法之后执行？此时就需要用到线程池的调度了，在调用 Release 之后，将会在某个 WaitAsync 方法，通过线程池分配线程继续执行。但是为了让线程池分配的线程知道是从哪里开始执行，就需要用到异步状态机了

在异步状态机记录当前方法上下文信息，而方法上下文信息是强引用的

看到这里，小伙伴也就知道我说的内存泄露的点在哪了

为了让 WaitAsync 方法能在 Release 之后，在线程继续执行，就需要通过异步状态机记录当前方法上下文信息，但是记录上下文信息就会存在强引用，而且这个引用是静态引用，也就是不会释放，因此如下代码会让对象不会释放

```csharp
    class Foo
    {
        public void F1()
        {
            _semaphoreSlim = new SemaphoreSlim(0);
            F2();
            _semaphoreSlim.Dispose();
            _semaphoreSlim = null;
        }

        private async void F2()
        {
            await _semaphoreSlim.WaitAsync();
            Console.WriteLine("F");
        }

        private SemaphoreSlim _semaphoreSlim;
    }
```

在调用 F1 方法的时候，将会使用 F2 方法等待 SemaphoreSlim 的释放，在 F2 的 WaitAsync 方法将会被异步状态机引用 Foo 对象

而在 F1 方法最后就干掉了 SemaphoreSlim 对象，此时再也不会有时机可以调用 Release 释放，此时异步状态机不会执行，也就是对 Foo 的引用不会释放，此时就存在内存泄露

我创建了两个 Foo 对象，一个调用了 F1 方法，另一个没有调用，然后放在弱引用对象里面，尝试调用垃圾回收之后判断对象是否存在，如下代码

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            F3();

            GC.Collect();
            GC.WaitForFullGCComplete();
            GC.Collect();

            Console.WriteLine(_weakReference1.TryGetTarget(out _));
            Console.WriteLine(_weakReference2.TryGetTarget(out _));

            Console.Read();
        }

        private static void F3()
        {
            _weakReference1 = new WeakReference<Foo>(new Foo());

            var foo = new Foo();
            _weakReference2 = new WeakReference<Foo>(foo);

            foo.F1();
        }

        private static WeakReference<Foo> _weakReference1;
        private static WeakReference<Foo> _weakReference2;
    }
```

可以看到输出是 

```csharp
False
True
```

也就是调用了 F1 方法的 Foo 对象不会被释放，而没有创建出来就放在 `_weakReference1` 的 Foo 对象被释放

因此我对官方的文档的说法有了理解 

```
Always call Dispose before you release your last reference to the SemaphoreSlim. Otherwise, the resources it is using will not be freed until the garbage collector calls the SemaphoreSlim object's Finalize method.
```

这是官方文档 [SemaphoreSlim.Dispose Method (System.Threading)](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim.dispose?view=netcore-3.1#System_Threading_SemaphoreSlim_Dispose )

这句话 `Always call Dispose before you release your last reference to the SemaphoreSlim` 的本质意思就是在调用 Dispose 之前需要编程开发者确保已经释放完成了所有的任务。同时官方文档也说到，调用 SemaphoreSlim 的 Dispose 方法不是线程安全的

因此安全的方式就是在调用 Dispose 之前先释放，干掉 WaitAsync 的逻辑，就如我在 [AsyncQueue](https://github.com/dotnet-campus/AsyncWorkerCollection/blob/8966732f996679fa6355554b61009099d744f4b4/AsyncWorkerCollection/AsyncQueue.cs) 写的代码

```csharp
        /// <summary>
        /// 主要用来释放锁，让 DequeueAsync 方法返回，解决因为锁让此对象内存不释放
        /// </summary>
        public void Dispose()
        {
            // 当释放的时候，将通过 _queue 的 Clear 清空内容，而通过 _semaphoreSlim 的释放让 DequeueAsync 释放锁
            // 此时将会在 DequeueAsync 进入 TryDequeue 方法，也许此时依然有开发者在 _queue.Clear() 之后插入元素，但是没关系，我只是需要保证调用 Dispose 之后会让 DequeueAsync 方法返回而已
            _isDisposed = true;
            _queue.Clear();
            // 释放 DequeueAsync 方法
            _semaphoreSlim.Release(int.MaxValue);
            _semaphoreSlim.Dispose();
        }
```

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/2f921b1c3a8c5b874f1e17ad8a53cae134432563/WayhemcurwelWemqonairbay)欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
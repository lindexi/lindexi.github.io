
# C# dotnet 自己实现一个线程同步上下文

昨天鹏飞哥问了我一个问题，为什么在控制台程序的主线程等待某个线程执行完成之后回来，是在其他线程执行的。而 WPF 在等待某个线程执行完成之后，可以回到主线程执行。其实这是因为在 WPF 和 WinForms 和 ASP.NET 框架里面都自己实现了线程同步上下文，通过线程同步上下文做到调度线程执行。本文就来和小伙伴聊一下如何自己实现一个线程同步上下文

<!--more-->


<!-- CreateTime:5/9/2020 8:22:22 AM -->



我昨天和鹏飞哥说的时候感觉特别绕，但是实际上过来写了一点代码，又发现很好理解。其实线程同步上下文这个概念在于我能否返回到之前的线程，返回到之前的线程需要哪些内容。而 `await` 在出现线程切换的时候，是通过调用之前等待之前的线程的线程同步上下文进行线程调度，大概在进入 await 的做法如下

```csharp
var currentSynchronizationContext = SynchronizationContext.Current;

// await 里面的复杂逻辑

   currentSynchronizationContext.Post(state =>
   {
      // 异步状态机调度过来的后面的任务
   }, state: null);
```

可以看到在 await 进入之前存放当前线程的同步上下文，而在执行完成之后，将后面的代码作为异步状态机调度创建委托，通过线程同步上下文的 Post 方法进行调度

那么什么是异步状态机调度过来的后面的任务，其实 await 只是语法加上很少的框架辅助做出来的，实际上代码就是通过一个个包装委托做的，如下面代码

```csharp
await Task.Run(() => {});
Foo();
```

此时的代码按照同步上下文的调用，可以在 IL 里面做如下的翻译

```csharp
// 在 await 任务之前先获取当前线程同步上下文
var currentSynchronizationContext = SynchronizationContext.Current;

Task.Run(() => {}).ContinueWith(t =>
{
      currentSynchronizationContext.Post(state =>
      {
          Foo();
	  }, state: null);
});
```

实际的 IL 会比上面代码复杂好多，原因是需要考虑存在多个不同的 await 以及不同的等待的内容的继续的写法，如 Task 通过调用 ContinueWith 方法在执行完成之后继续

从上面代码可以看到实际上线程同步上下文只是执行 await 后面的代码的方法，如果在调用 currentSynchronizationContext.Post 能让传入的委托在原有线程执行是不是就和 WPF 等框架相同

实际上 WPF 大概也是这样写的，下面来写一个自定义的线程同步上下文，让主线程加上线程同步上下文做到在等待其他线程执行完成返回可以到主线程执行

```csharp
class SycnContext : SynchronizationContext
```

在继承了SynchronizationContext类，可以重写两个主要的方法，就是 Post 和 Send 方法。这两个方法的含义就是 Post 就是调用方不等待调用的内容执行完成，调用只是让他执行，不等待执行完成。而 Send 就是调用方需要等待 Send 传入的委托执行完成

```csharp
        public override void Post(SendOrPostCallback d, object state)
        {
        }

        /// <inheritdoc />
        public override void Send(SendOrPostCallback d, object state)
        {
          
        }
```

这就是两个关键方法的重写，而默认的 SynchronizationContext 是如何实现的？ 请看开源的 [源代码](https://github.com/dotnet/runtime/blob/e77572ffccc566186f47207f3c5475533c87538e/src/libraries/System.Private.CoreLib/src/System/Threading/SynchronizationContext.cs#L23-L25) 实际上十分简单

```csharp
        public virtual void Send(SendOrPostCallback d, object? state) => d(state);

        public virtual void Post(SendOrPostCallback d, object? state) => ThreadPool.QueueUserWorkItem(s => s.d(s.state), (d, state), preferLocal: false);
```

可以看到默认的 Post 是通过线程池的方式调用，这就是为什么回不到主线程的原因

那么在重写这个方法如何让调用的内容回到主线程执行？回到主线程执行有前提是主线程需要有空，如果主线程没有空那么如何执行。从方法上传入的只是一个委托，如何让这个委托在主线程执行。这需要主线程主动去执行才可以

在 SycnContext 类添加一个锁，然后主线程空闲的时候就等待这个锁。而在有代码调用 Post 方法的时候，就释放这个锁，让主线程执行调用进来的委托

```csharp
        public override void Post(SendOrPostCallback d, object state)
        {
            Run = () => d(state);
            Event.Set();
        }

        public Action Run { private set; get; }

        public AutoResetEvent Event { get; } = new AutoResetEvent(false);
```

上面代码的锁用的是 AutoResetEvent 类，这个类的功能就是在调用 WaitOne 的时候进入锁等待，直到其他线程调用了 Set 方法才会继续执行

在主线程可以等待 AutoResetEvent 如果等待返回了就执行 Run 委托

```csharp
            var synchronizationContext = new SycnContext();

            while (true)
            {
                synchronizationContext.Event.WaitOne();
                synchronizationContext.Run();
            }
```

那么如何让 await 在执行之前可以拿到线程同步上下文？可以通过 SynchronizationContext 的一个静态方法设置线程静态字段

```csharp
            var synchronizationContext = new SycnContext();
            
            SynchronizationContext.SetSynchronizationContext(synchronizationContext);
```

上面涉及到一个概念是线程静态字段，什么是线程静态字段，和静态字段有什么不同？在 dotnet 里面的静态字段是所有线程访问到的对象都是相同的对象。而线程静态字段是只有相同的线程才能访问到相同的对象，不同的线程访问到的是不同的对象。而上面代码是将线程同步上下文设置到当前的线程的一个线程静态字段里面，也就是在当前线程访问的线程同步上下文都是刚才设置的对象，但其他线程访问的是其他对象

请看官方的代码在获取当前线程同步上下文的代码

```csharp
public static SynchronizationContext? Current => Thread.CurrentThread._synchronizationContext;
```

小伙伴都用过 Thread.CurrentThread 这个静态属性，这个属性返回的就是当前线程，也就是不同的线程拿到的对象是不同的。更多线程静态请看 [dotnet 线程静态字段](https://blog.lindexi.com/post/dotnet-%E7%BA%BF%E7%A8%8B%E9%9D%99%E6%80%81%E5%AD%97%E6%AE%B5.html )

现在添加一个等待后台线程的代码

```csharp
        private static async void Foo()
        {
            var task = Task.Run(async () =>
            {
                await Task.Delay(100);
                Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}");
            });
            await task;
            Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}");
        }
```

在这个方法里面输出了当前的线程是哪个

现在的主函数如下

```csharp
        static void Main(string[] args)
        {
            var synchronizationContext = new SycnContext();
            
            SynchronizationContext.SetSynchronizationContext(synchronizationContext);
         
            Console.WriteLine($"{Thread.CurrentThread.ManagedThreadId}");
            Foo();

            while (true)
            {
                synchronizationContext.Event.WaitOne();
                synchronizationContext.Run();
            }
        }
```

在运行的时候推荐添加断点，在 `Foo();` 添加断点，在 `while (true)` 添加断点，在 `await task;` 添加断点，这样小伙伴就可以看到调用的顺序了

在调用 Foo() 方法进入到 `await task;` 方法的时候，主线程执行到 `await` 就出让执行权，返回到 Foo 外面，执行 `while (true)` 代码。在 Task 里面执行到了 `await Task.Delay(100);` 完成，再到输出当前线程是哪个之后，将会完成 `await task;` 的代码，此时将会通过 SynchronizationContext 的 Post 方法将后面的输出作为委托传入

在 Post 方法里面将会先设置 Run 委托，然后释放锁让主线程继续执行，主线程将会执行 Run 委托，也就是执行 `await task;` 之后的代码

因为是主线程执行 `await task;` 之后的代码，所以效果就是等待线程返回之后回到主线程继续执行

刚才的代码还少了 Send 方法，其实 Send 方法就是需要在执行完成传入的委托才能返回，可以通过一个锁来做

```csharp
        public override void Send(SendOrPostCallback d, object state)
        {
            // 用于了解执行完成
            AutoResetEvent autoResetEvent = new AutoResetEvent(false);
            Run = () =>
            {
                d(state);
                autoResetEvent.Set();
            };
            Event.Set();
            autoResetEvent.WaitOne();
        }
```

那在 WPF 是如何实现的？其实 WPF 有一个 DispatcherSynchronizationContext 类，逻辑和上面自定义的差不多，请看[源代码](https://github.com/dotnet/wpf/blob/75126b39e66a9d99cd0dd30bc9abe314209b5190/src/Microsoft.DotNet.Wpf/src/WindowsBase/System/Windows/Threading/DispatcherSynchronizationContext.cs) 代码核心通过 Dispatcher 实现

说起来也许复杂，但是写一写就知道是怎么弄的


本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ada1cf4b994a20395071af3c806f7a3a7dda41fb/CallnernawbawceKairwemwhejeene) 欢迎小伙伴访问

在 WPF 里面是如何做的？请看 [异步函数async await在wpf都做了什么？ - RyzenAdorer - 博客园](https://www.cnblogs.com/ryzen/p/13062963.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
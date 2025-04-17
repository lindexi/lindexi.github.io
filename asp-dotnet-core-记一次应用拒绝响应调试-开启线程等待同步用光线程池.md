
# asp dotnet core 记一次应用拒绝响应调试 开启线程等待同步用光线程池

我有一个上古的库，我使用这个库用来上报日志，而刚才日志服务挂了。然后我就发现了我的应用拒绝响应了，通过 VisualStudio 断点调试可以发现线程池的线程全部被占用了。因为没有可用线程因此所有对 asp dotnet core 应用的访问全部都不会收到响应，为什么我的另一个应用日志服务挂了会让我的业务应用拒绝响应？为什么我的业务应用会使用线程池所有的线程，为什么线程池的所有线程被占用将会让应用拒绝响应

<!--more-->


<!-- CreateTime:2020/9/22 8:30:24 -->



很好复现这个坑，在开始复现之前，需要聊一下背景

我有一个业务应用和一个日志服务，基本上可以认为日志服务和业务没有任何关联，而且我从上层业务调用可以看到，都是异步使用。而在日志服务全部挂掉的时候，开始业务应用还能使用，但是当请求大概访问了 100 次，就发现后续的访问都没有任何返回。同时在业务应用的本机控制台和日志文件里面都没有任何记录，而控制台也没有收到 50x 等错误，也就是业务应用还在工作，但是没有任何响应


我在本地上可以复现，使用 VisualStudio 开启所有异常，也什么都没收到。在应用配置文件 appsettings.json 文件里面将日志配置设置为 Debug 也没有拿到任何有用的信息

原本每次的请求都会在默认的 asp dotnet core 日志输出至少一条日志，但是此时什么日志都没有输出

而此时的业务应用的 cpu 和内存占用都很少，在没有请求的时候，可以看到 cpu 几乎没有占用

在点击 VisualStudio 暂停的时候，可以看到业务应用创建了大量的线程

![](http://cdn.lindexi.site/lindexi%2F2020921211113804.jpg)

其实调试到线程的时候，大概半个下午了，哈哈

其实我不知道如果一个 asp dotnet core 应用对所有的请求都没有返回，也没有报错的时候可以如何调试

在看到有大量的线程被创建的时候，此时可以调试的是打开 调试->窗口->并行堆栈 这个工具可以辅助调查所有线程问题

如果一个应用创建了大量线程，如果这些线程都是通过 Task.Run 创建，那么意味着线程池里面的线程全部都使用了。也就是此时的下一次调用 Task.Run 需要等待线程池重新分配或创建线程。如果线程池没有空闲的可以分配需要等待一段时间才能创建新的线程，于是此时的应用就会卡住没有返回值

而根据 Eleven 老师的 asp dotnet core 源代码分析课程可以了解到，在 asp dotnet core 服务主机里面的线程是主线程固定的，但是调用到对应的控制器需要通过线程池调度。当然更多细节还请小伙伴关注 Eleven 老师的社区

![](http://cdn.lindexi.site/lindexi%2F20209121930471745.jpg)

在用光线程池的线程，此时的请求可以被主机处理，因此不会抛出远程服务器拒绝请求。但是主机通过线程池调度到对应控制器，因为线程池没有足够的线程，因此将会进入很长的等待。特别是有后续请求，那么将需要不断排队。这就是为什么我看到的业务应用拒绝服务

进一步的调试可以通过并行堆栈找到最多相同的堆栈，也就是有多少线程都在相同的堆栈里，那么证明这部分逻辑有锅

我在调试中看到如下代码

![](http://cdn.lindexi.site/lindexi%2F20209212123591748.jpg)

我的底层库给我的方法是异步的上报日志方法，但是这个日志上报方法的核心是通过 Task.Run 一个线程进行同步调用

其实在 asp dotnet core 的性能优化中，要尽量不使用 Task.Run 方法，在 [ASP.NET Core Performance Best Practices 官方文档](https://docs.microsoft.com/en-us/aspnet/core/performance/performance-best-practices?view=aspnetcore-3.1) 和译文 [ASP.NET Core 性能优化最佳实践 - Newbe36524 - 博客园](https://www.cnblogs.com/newbe36524/p/13663722.html) 都有提到，原因还请小伙伴看这两篇博客

![](http://cdn.lindexi.site/lindexi%2F20209212126156930.jpg)

那么为什么上面的代码将会让线程池的线程都在等待？原因是 GetResponse 是一句同步的代码，同步的代码等待网络的返回，而此时我的日志服务大概写了如下代码

```csharp
        [HttpGet]
        [Route("/")]
        public async Task<IActionResult> Get()
        {
            await Task.Task;

            return Ok();
        }

        private static readonly TaskCompletionSource<bool> Task = new TaskCompletionSource<bool>();
```

这是我写出的最简单的日志服务的代码，这个代码的所有请求都会进入到 `await Task.Task;` 等待一个不会返回的任务，也就是任何的请求进来只能等待超时

而刚好上面业务应用的等待是没有设置超时的，在同步的调用等待一个不会返回的请求，此时的线程就被占用了

如果业务应用对每次请求都需要进行如上面的从线程池获取线程然后进行同步访问，那么线程池的将会被用光。在线程池的线程都被占用的时候，下次调用 Task.Run 就会先等待一段时间，如果等待一段时间还没有线程可以调度，那么此时才会在线程池新建线程

所以应用如果拒绝响应，首先需要调查应用是否用光了线程池，然后再调查连接数。如果是线程池用光，那么打开并行堆栈，看线程最多的堆栈是什么，然后通过堆栈和源代码可以找到是否存在锁或者调用 IO 同步

如果发现这个的 asp dotnet core 应用的性能不足，因为线程开启过多，那么此时可以全局找 Task.Run 的代码，尽可能干掉这部分逻辑

而本文的坑，可以使用将同步修改为异步的方法解决，换句话说，不需要通过线程池开启线程的方法，通过IO自带的异步方法进行异步IO请求。此时在 IO 的异步里面将会自动出让 CPU 执行，这部分是硬件的支持，因此进入异步的 IO 将不会占用线程，线程可以回到线程池给其他业务调用

一个可选的方法是将一些不重要但是需要慢慢执行的任务放在生产者消费者队列里面，如果这部分任务很小，可以尝试使用我的 AsyncQueue 高性能低资源占用的类，详细请看 [dotnet 使用 AsyncQueue 创建高性能内存生产者消费者队列](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-AsyncQueue-%E5%88%9B%E5%BB%BA%E9%AB%98%E6%80%A7%E8%83%BD%E5%86%85%E5%AD%98%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85%E9%98%9F%E5%88%97.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
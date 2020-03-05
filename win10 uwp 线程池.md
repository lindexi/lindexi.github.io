# win10 uwp 线程池

如果大家有开发 WPF 或以前的程序，大概知道线程池不是 UWP 创造的，实际上在很多技术都用到线程池。

为什么需要线程池，他是什么？如何在 UWP 使用线程池，本文就是来告诉大家这些

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

<!-- csdn -->
<div id="toc"></div>
<!-- 标签：UWP，多线程 -->

## 为什么需要线程池

在程序中，创建和销毁线程是需要很多资源的，如果只是为了完成很小的代码而创建一个新的线程，创建线程占用的时间在总的运行时间占有比例很大。所以大神就说，那就不创建线程了。

因为花费总时间划不来，所以大神就想直接创建一个线程，也不销毁，一旦用户需要一个新线程去做一些事情，就把这个线程给他。这样就可以省略了创建和销毁线程时间，减少了花费总时间。

## 什么是线程池

百度说线程池是一种多线程处理形式，处理过程中将任务添加到队列，然后在创建线程后自动启动这些任务。

在 C# 中，线程池只是预先分配了一些线程，线程没事做就休息，有工作需要就随便叫一个线程出来。通过这个方法减少创建线程的时间。
 
## 线程池原理

在 C# 大家都知道，执行一个方法，如果需要把方法传到另一个时间去调用，就可以使用委托。而创建一个线程去做其他的事情，实际上可以认为是把一个委托传入一个线程，让这个线程使用。线程池就是先创建了很多线程，用户调用就是传入方法，线程池拿出一个空闲的线程去执行传入的方法。

最简单的模拟代码就是创建一个线程，然后让他运行一个委托，运行完成设置这个委托为空。

```csharp
        private Action _action;

            new Thread(() =>
            {
                while (true)
                {
                    _action?.Invoke();
                    _action = null;
                }
            }).Start();
```

上面代码是无法在 UWP 运行的，只是告诉大家原理。
 
因为做这个线程池需要很多代码，如判断设备运行多少个线程合适，分配空闲线程等。好像微软已经弄好了，大家只需要用。

## 应用

大家从原理可以知道，线程池运行代码，不是立刻运行的，假如线程池有10个线程，刚好都在做其他事情，这时请线程池运行新的代码，就会等待线程池存在空闲线程。

千万不要使用线程池执行比较紧急的任务，因为可能等待很多时间都没运行。

在 UWP 可以通过 ThreadPool 使用线程池。

```csharp
            var thread = ThreadPool.RunAsync(e =>
            {
                Debug.WriteLine("使用线程池很简单");
            });
```

使用的方法十分简单，传入一个委托就可以。

## 等待代码完成

很多时候的线程模型就是需要运行很多并行代码，在运行完成再运行串行的代码。

![](http://image.acmx.xyz/lindexi%2F2018515193336896.jpg)

这时就需要使用线程池运行代码，还需要等待代码运行完成

例如我需要下载 lindexi.github.io 所有博客，获得所有文章只能使用一个线程获取，但是下载所有博客就可以并行。在所有下载完成还需要告诉用户，这时只能使用一个线程。

```csharp
            var url = "lindexi.github.io";

            // 获取所有文章
            var aritcle = GetArticle(url);

            var threadList = new List<IAsyncAction>();

            foreach (var temp in aritcle)
            {
                threadList.Add(ThreadPool.RunAsync(e =>
                {
                    Download(temp);
                }));
            }

            Task.WaitAll(threadList.Select(temp => temp.AsTask()).ToArray());

            Debug.WriteLine("下载完所有博客");
```

## 定时器

如果需要一个定时器，除了使用主线程的定时器，还可以使用 ThreadPoolTimer ，创建一个定时器很简单，请看代码

```csharp
            ThreadPoolTimer.CreateTimer(timer =>
            {
                Debug.WriteLine("下载完所有博客");
            }, new TimeSpan(0, 0, 0, 1));
```

实际上从效果，可以把上面的代码认为是

```csharp
            Task.Delay(new TimeSpan(0, 0, 0, 1)).ContinueWith(_ =>
            {
                Debug.WriteLine("下载完所有博客");
            });
```

如果需要重复执行，请使用 CreatePeriodicTimer ，这个函数可以延迟大于指定的时间执行代码

```csharp
            ThreadPoolTimer.CreatePeriodicTimer(timer =>
            {
                Debug.WriteLine("下载完所有博客");
            }, new TimeSpan(0, 0, 0, 1));
```

如果想让软件代码不会被大神喷，那么请看一下[使用线程池的最佳做法](https://docs.microsoft.com/zh-cn/windows/uwp/threading-async/best-practices-for-using-the-thread-pool )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

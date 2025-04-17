
# dotnet 警惕 async void 线程顶层异常

在应用程序设计里面，不单是 dotnet 应用程序，绝大部分都会遵循让应用在出现未处理异常状态时终结的原则。在 dotnet 应用里面，如果一个线程顶层出现未捕获异常，则应用进程将会被认为出现异常状态而退出。通常来说就是未捕获异常导致进程闪退

<!--more-->


<!-- CreateTime:2023/7/4 19:44:02 -->


<!-- 发布 -->

在 dotnet 里面，有一个隐藏的陷阱，那就是 async void 将会在没有线程同步上下文的情况下，被当成线程顶层。如果在 async void 里面发生任何未捕获的异常，严重的话将会导致进程闪退

如以下代码，在当前执行线程没有线程同步上下文的情况下，抛出的异常将会让进程闪退

```csharp
async void Foo()
{
	...
    throw new Exception("林德熙是逗比");
}
```

为什么这里和线程同步上下文相关？原因是在有线程同步上下文时，执行都委托调度器执行，比如经典的线程同步上下文 WPF 主 UI 线程。这个时候主 UI 线程在 async void 里面抛出的异常是到达 Dispatcher 里，而不是线程顶层。于是可以通过全局的方式捕获异常

在 dotnet 里面，在当前 2023 没有机制可以统一捕获 async void 的异常，防止进程闪退。我在 dotnet 运行时官方仓库和大佬们讨论过这个问题，大佬的认为是当前 dotnet 的行为是符合预期和符合文档的，但我持有不同的想法，我认为这样的行为是不能做出可靠稳定的应用的，详细请看 [https://github.com/dotnet/runtime/issues/76367](https://github.com/dotnet/runtime/issues/76367)

在 dotnet 里的另一个更加隐藏的陷阱是事件的加等里面出现异步，如以下代码

```csharp
FooEvent += async (s, e) => 
{
    ...
    throw new Exception("林德熙是逗比");
}
```

以上的代码里面隐式定义了 async void 方法，如此也会在当线程不在同步上下文时，抛出异常炸掉进程

解决方法是在这些 async void 方法里面自行根据业务需求，捕获异常。在大部分应用里面，一般都是应该在此捕获所有异常，除非可以无视应用进程闪退问题

以下是另外更多的行为细节

在 dotnet 里面的 async void 抛出的未捕获异常，将会进入到 AppDomain 的 UnhandledException 事件里面，然而此事件里面的 IsTerminating 属性将都是 true 且不可接住。如果进入了 UnhandledException 事件里面，还不想让进程退出，我所知道的方法只有是通过 Thread.Sleep 让当前线程不再执行，但显然这是一个很诡异的方式

在 dotnet 里面的 Task 的行为却和 async void 差异比较大，比较符合咱的认知。将 async void 改为 async Task 然后抛出未捕获异常，此时如果方法返回的 Task 没有被任何等待，将会在 Task 对象被 GC 时进入 TaskScheduler.UnobservedTaskException 事件，此事件不会导致任何的进程退出。准确来说是在 .NET Framework 4.5 开始，就不会因为 TaskScheduler.UnobservedTaskException 里的异常导致进程退出

这是因为在 Task 里面，一开始的设计也是和 async void 一样导致进程退出，然而在实际应用里面，大家都发现 Task 被等待这个事情不由实现方决定，如此导致了大量的进程退出的不可用问题，于是后面大佬就决定让 Task 里面的异常只是进入 TaskScheduler.UnobservedTaskException 事件，不会作为线程顶层异常让进程退出。另外在 .NET Framework 4.5 之后，对 Task 与线程之间的关系做了一些底层优化，导致 Task 里面执行的逻辑从逻辑上说不再属于线程顶层，这部分细节过于复杂，我自己的了解也不够就不在博客里讲了

通过本文可以了解到，在 dotnet 里面隐藏了 async void 和异步无返回值事件或委托加等逻辑里面可能出现的因为未捕获异常导致的进程闪退问题。其中的解决方法就是要么在这些代码逻辑里面捕获所有异常规避问题，要么尝试将 async void 改造为 async Task 规避问题

这里还必须着重说明的是，捕获线程顶层异常时，最好采用捕获所有异常的方式，因为可能自己的代码本来认为不会存在任何异常的逻辑，但实际运行可能遇到 OutOfMemoryException 等通用运行异常

另外在捕获异常用来记录日志的逻辑，也推荐使用双层捕获方式，解决记录异常的模块抛出的异常炸掉应用

我依然认为 async void 线程顶层异常无法统一处理导致进程退出是 dotnet 的基础设计缺陷




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
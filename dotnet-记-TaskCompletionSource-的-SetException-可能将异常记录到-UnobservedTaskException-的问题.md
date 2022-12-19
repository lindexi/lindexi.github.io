
# dotnet 记 TaskCompletionSource 的 SetException 可能将异常记录到 UnobservedTaskException 的问题

本文将记录 dotnet 的一个已知问题，且是设计如此的问题。假定有一个 TaskCompletionSource 对象，此对象的 Task 没有被任何地方引用等待。在 TaskCompletionSource 被调用 SetException 或 TrySetException 方法时，将会记录一个存在异常且未捕获的 Task 对象。此 Task 对象将会在被 GC 回收时，进入 TaskScheduler.UnobservedTaskException 事件里面，尽管没有明确副作用，但是会吓到不明真相的开发者

<!--more-->


<!-- CreateTime:2022/12/16 8:31:50 -->

<!-- 博客 -->
<!-- 发布 -->

最简的复现步骤是如下代码

```csharp
    public App()
    {
        TaskScheduler.UnobservedTaskException += TaskSchedulerOnUnobservedTaskException;

        var taskCompletionSource = new TaskCompletionSource();
        taskCompletionSource.SetException(new Exception());
    }

    private void TaskSchedulerOnUnobservedTaskException(object? sender, UnobservedTaskExceptionEventArgs e)
    {
    }
```

以上是一个 WPF 应用，选 WPF 应用是可以比较方便等待 GC 的触发

以上代码将创建一个 TaskCompletionSource 对象，且此对象的 Task 没有地方等待，意味着在 SetException 设置的异常，将会设置到一个未等待的 Task 上。按照 dotnet 的设计，如果一个 Task 存在未捕获的异常，将会在 Task 被回收的时候，进入 TaskScheduler.UnobservedTaskException 事件

进入 TaskScheduler.UnobservedTaskException 事件的异常，按照设计，在 .NET Framework 4.5 之后，也就是包含所有的 dotnet core 版本，都不会导致应用崩溃。进入 TaskScheduler.UnobservedTaskException 事件的异常，只是用来告诉开发者，某个地方也许存在一个小坑

规避方法是： 无视；找个好地方等待一下 Task 对象；不要 SetException 方法，而是换成 SetResult 方法




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
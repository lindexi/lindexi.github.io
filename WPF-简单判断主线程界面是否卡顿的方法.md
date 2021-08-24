
# WPF 简单判断主线程界面是否卡顿的方法

本文来告诉大家如何使用简单的代码判断当前的软件的 UI 线程或界面是否卡顿

<!--more-->


<!-- 博客 -->
<!-- 发布 -->

在后台线程调用如下代码即可用来判断是否卡顿

```csharp
        private static async Task<bool> CheckDispatcherHangAsync(Dispatcher dispatcher)
        {
            var taskCompletionSource = new TaskCompletionSource<bool>();
            _ = dispatcher.InvokeAsync(() => taskCompletionSource.TrySetResult(true));
            await Task.WhenAny(taskCompletionSource.Task, Task.Delay(TimeSpan.FromSeconds(2)));
            // 如果任务还没完成，就是界面卡了
            return taskCompletionSource.Task.IsCompleted is false;
        }
```

以上代码的 Dispatcher 可以从窗口里面获取，也可以使用 Application.Current.Dispatcher 获取。以上代码推荐在后台线程调用

原理就是给 UI 线程一个任务，如果此任务可以在 2 秒执行完成，那么就证明 UI 不卡，否则就是证明 UI 卡了

以上方法可以放在后台线程的循环进行不断调用，但是多次调用之间需要等待一下，不然将会让 UI 线程太忙





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
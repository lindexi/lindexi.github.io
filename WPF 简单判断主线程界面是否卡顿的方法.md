# WPF 简单判断主线程界面是否卡顿的方法

本文来告诉大家如何使用简单的代码判断当前的软件的 UI 线程或界面是否卡顿

<!--more-->
<!-- CreateTime:2021/8/23 16:49:17 -->

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

另外，利用此思想，还可以用来测量渲染间隔。做法是在 InvokeAsync 里面设置使用 Render 优先级，如以下代码

```csharp
            var currentTime = Environment.TickCount64;

            Dispatcher.InvokeAsync(() =>
            {
                var delay = Environment.TickCount64 - currentTime;
                // 这里的 delay 就是距离评估时的延迟。可以多次调用获取平均值

            }, DispatcherPriority.Render);
```

实现的原理就是传入 Render 优先级，看在 Render 优先级下需要多久才能处理一次任务，从而间接了解到渲染间隔

但是必须需要说明的是，采用此方法不能用来测量画面渲染频率。只能用来做粗糙的渲染评估，用此方法评估虽然不准，但是胜在评估的性能极好，对整体的性能影响极小

毕竟要是评估模块本身就影响性能，那还玩什么呢？

为什么说采用此方式测量只能做粗糙的渲染评估？了解这个问题的原因之前，还需要简单了解一下 WPF 的渲染线程架构。在 WPF 里面，有一条 UI 线程和一条渲染线程。在 UI 线程里面，也就是咱刚才写的 Dispatcher 执行的逻辑，是在 UI 线程上执行的，而不是真正处理渲染的渲染线程上执行的。从这个角度上也可以了解到这个评估不是能够直接评估渲染频率的方法

但是 UI 线程是会等待渲染线程的执行结果的，同时渲染线程要干活，必须要有 UI 的指导。如果 UI 线程不告诉渲染线程需要渲染什么内容，那自然渲染线程也不能渲染内容。而 UI 线程要告诉渲染线程干什么，框架内是在 Render 优先级的任务告诉的渲染线程的。因此通过注入一条 Render 优先级的任务，即可评估出粗糙的渲染间隔。多次取平均值，能够了解渲染的延迟

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

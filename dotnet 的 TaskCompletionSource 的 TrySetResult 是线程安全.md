# dotnet 的 TaskCompletionSource 的 TrySetResult 是线程安全

在创建一个 TaskCompletionSource 期望让等待的逻辑只会被调用一次，而调用的是多线程，可以使用 TrySetResult 方法，这个方法是线程安全，只会让 TaskCompletionSource 被调用一次

<!--more-->
<!-- CreateTime:2020/7/27 19:52:44 -->

<!-- 发布 -->

在多个线程调用 TaskCompletionSource 的 TrySetResult 方法，只有一个线程能进入设置，其他线程将会拿到返回 false 的值

测试代码如下

```csharp
            _taskCompletionSource = new TaskCompletionSource<bool>();

            Foo();

            var taskList = new List<Task>();

            for (int i = 0; i < 100; i++)
            {
                var task = Task.Run(() =>
                {
                    _taskCompletionSource.TrySetResult(true);
                });

                taskList.Add(task);
            }

            Task.WaitAll(taskList.ToArray());

            Console.Read();

        private static async void Foo()
        {
            await _taskCompletionSource.Task;
            Console.WriteLine("F");
        }

        private static TaskCompletionSource<bool> _taskCompletionSource;
```

可以看到使用很多线程调用，而 Foo 只执行一次输出

上面测试的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/916a11d68c0fba14c75b5e438bdecddb1ff421be/CallbadojuBaheanurjair ) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


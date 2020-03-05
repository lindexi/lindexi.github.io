# C＃同步方法转异步

本文来告诉大家如何把同步的代码转换为异步

<!--more-->
<!-- CreateTime:2019/9/2 12:57:37 -->


<div id="toc"></div>

## 创建新线程

最简单的方法是创建一个新的线程，创建的方法是使用 Task.Run ，请看下面代码，原来有一个函数 Delay 需要把他转换为异步，就可以使用 DelayAsync 里面用一个线程

```csharp
  
  public async Task DelayAsync()
  {
  
      await Task.Run(()=>Delay());
  }

  private void Delay()
  {


  }
```

## AMP 转 EAP

在很古老的开发，微软告诉大家使用 AMP 异步编程模型 Asynchronous Programming Model，这个模型就是使用 Begin xx 和 End xx 的方法 如 FileStream.BeginRead 和Stream.EndRead 表示

现在微软告诉大家，建议使用 EAP 基于事件的异步模式，也就是 Async 的模型

例如有一个文件读取，文件读取可以使用 BeginRead 和 EndRead ，看到下面代码大家就会觉得这很难用

```csharp
            var fileStream = new FileStream("E:\\lindexi\\博客",FileMode.Open);

            var buffer =new byte[65535];
            IAsyncResult asyncResult=null;
            fileStream.BeginRead(buffer, 0, buffer.Length, ar => { asyncResult = ar;}, null);

            fileStream.EndRead(asyncResult);
```

好在微软提供一个方式把上面的代码转换为 async ，方法是 Task.Factory.FromAsync 请看代码

```csharp
await Task.Factory.FromAsync(fileStream.BeginRead, fileStream.EndRead, buffer, 0, buffer.Length, null);
```

如果希望对于如动画的等待，那么建议看[如何实现一个可以用 await 异步等待的 Awaiter - walterlv](https://walterlv.github.io/post/write-custom-awaiter.html )

从[如何实现一个可以用 await 异步等待的 Awaiter - walterlv](https://walterlv.github.io/post/write-custom-awaiter.html )复制出来类 [DispatcherAsyncOperation](https://github.com/walterlv/sharing-demo/blob/master/src/Walterlv.Demo.Sharing/Utils/Threading/DispatcherAsyncOperation.cs)

动画的等待是在动画结束返回函数，也许这里比较难说，动画有开始和结束事件，希望在结束事件函数才返回

```csharp
       private void TrirlelJallardra()
        {
            // 执行动画
            TeedraiseTeretal();

            // 期望在动画结束才做的代码

        }

        /// <summary>
        /// 执行动画
        /// </summary>
        private void TeedraiseTeretal()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回
            };
        }
```

因为 TrirlelJallardra 拿不到 storyboard.Completed 所以如果要在动画完成之后写调用代码是很难的。

为什么需要在其他函数写？如果是继续执行动画，而且需要在上一个动画执行完成，写在 Completed 的代码会很多。如果可以使用下面的函数的方法，可读性比较好

```csharp
      private void TrirlelJallardra()
        {
            // 执行动画
            Animation1();

            // 期望在动画1完成继续动画2
            Animation2();

            // 在上一个动画完成才调用下一个动画

            Animation3();

        }

        /// <summary>
        /// 执行动画1
        /// </summary>
        private void Animation1()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回
            };
        }

        /// <summary>
        /// 执行动画2
        /// </summary>
        private void Animation2()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回
            };
        }

        /// <summary>
        /// 执行动画3
        /// </summary>
        private void Animation3()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回
            };
        }
```

如果需要写在回调里，那么可读性比较差

```csharp
     private void TrirlelJallardra()
        {
            // 执行动画
            Animation1();
        }

        /// <summary>
        /// 执行动画1
        /// </summary>
        private void Animation1()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回

                // 期望在动画1完成继续动画2
                Animation2();

            };
        }

        /// <summary>
        /// 执行动画2
        /// </summary>
        private void Animation2()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回


                // 在上一个动画完成才调用下一个动画

                Animation3();
            };
        }

        /// <summary>
        /// 执行动画3
        /// </summary>
        private void Animation3()
        {
            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回
            };
        }
```

那么这时使用 DispatcherAsyncOperation 就可以做异步，让代码可读性比上面好

```csharp
     private async void TrirlelJallardraAsync()
        {
            // 执行动画
            await Animation1();

            // 期望在动画1完成继续动画2
            await Animation2();

            // 在上一个动画完成才调用下一个动画

            await Animation3();
        }

        /// <summary>
        /// 执行动画1
        /// </summary>
        private DispatcherAsyncOperation<bool> Animation1()
        {
            var dispatcherAsyncOperation = DispatcherAsyncOperation<bool>.Create(out var action);

            var storyboard = new Storyboard();

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) =>
            {
                // 这时函数才返回
                action(true, null);
            };

            return dispatcherAsyncOperation;
        }

        /// <summary>
        /// 执行动画2
        /// </summary>
        private DispatcherAsyncOperation<bool> Animation2()
        {
            var storyboard = new Storyboard();

            var dispatcherAsyncOperation = DispatcherAsyncOperation<bool>.Create(out var action);

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) => { action(true, null); };

            return dispatcherAsyncOperation;
        }

        /// <summary>
        /// 执行动画3
        /// </summary>
        private DispatcherAsyncOperation<bool> Animation3()
        {
            var storyboard = new Storyboard();

            var dispatcherAsyncOperation = DispatcherAsyncOperation<bool>.Create(out var action);

            //设置动画

            storyboard.Begin(this);

            storyboard.Completed += (sender, args) => { action(true, null); };

            return dispatcherAsyncOperation;
        }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
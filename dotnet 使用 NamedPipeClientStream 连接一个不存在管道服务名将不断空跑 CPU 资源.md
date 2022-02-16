# dotnet 使用 NamedPipeClientStream 连接一个不存在管道服务名将不断空跑 CPU 资源

本文记录一个开发和代码审查过程中，需要关注的细节。在 dotnet 里，在 .NET 6 和以下版本，包括 .NET Framework 版本，使用 NamedPipeClientStream 进行连接管道服务，如果此时的管道服务没有存在，或者还没有启动，调用 ConnectAsync 或 Connect 方法，将会进入一个循环，不断进行空跑，等待超时或者是连接上。默认的 ConnectAsync 或 Connect 方法，传入的超时时间都是无穷，也就是将会无限重试，不断消耗 CPU 资源

<!--more-->
<!-- CreateTime:2022/2/12 10:41:44 -->

<!-- 发布 -->
<!-- 博客 -->

咱可以使用 NamedPipeClientStream 去连接一个管道服务，从而建立多进程之间的通讯。在连接时，最好是先有管道服务启动，然后再启动管道客户端 NamedPipeClientStream 进行连接。因为如果在 NamedPipeClientStream 开始 Connect 时，还不存在管道服务，那将有一段时间进行 CPU 的空跑

不过好在 Connect 底层实现上，采用了 SpinWait 的 SpinOnce 方法进行自旋，此自旋是一个混合自旋方式，当次数多的时候，将会自动出让 CPU 执行权。但是如果等待连接的数量足够多，依然会占用一定的 CPU 资源，占用多少，取决于 CPU 的价格（价格约等于性能）哈。如以下的代码，将会不断去连接一个不存在的管道名

```csharp
using System.IO.Pipes;
using System.Security.Principal;

for (int i = 0; i < 1000; i++)
{
    var namedPipeClientStream = new NamedPipeClientStream(".", "NotExists_" + i, PipeDirection.Out,
        PipeOptions.None, TokenImpersonationLevel.Impersonation);
    // Task.Factory.StartNew(namedPipeClientStream.Connect, TaskCreationOptions.LongRunning);
   _ = namedPipeClientStream.ConnectAsync();
}

Console.Read();
```

尝试运行以上的代码，可以看到 CPU 将会不断上升。使用 ConnectAsync 版本，线程数量上升较慢，同时 CPU 上升速度也较慢。如使用被注释的 `Task.Factory.StartNew(namedPipeClientStream.Connect, TaskCreationOptions.LongRunning)` 代码，那可以看到 CPU 将会快速被占用，线程也有大量的数量

因此在开发的时候，如果需要使用 NamedPipeClientStream 进行 Connect 或 ConnectAsync 连接，除非能明确管道的服务端已创建成功，否则都推荐加上超时逻辑。不然，在尝试连接一个不存在的服务管道名，将会占用线程，不断空跑。数量少的时候，没有什么影响，数量多的时候，将会浪费 CPU 资源

如果关心 .NET 的底层实现，为什么会有此问题，请继续阅读

在 .NET 6 和以下版本，包括 .NET Framework 版本，使用 NamedPipeClientStream 的 ConnectAsync 方法，本质上相当于使用 `Task.Run` 包一个 Connect 方法，如以下的 .NET 6 有删减的代码。为了让本文清晰，本文以下就只讨论使用 Connect 方法的逻辑

```csharp
    public sealed partial class NamedPipeClientStream : PipeStream
    {
    	// 为了让文章清晰，删减部分代码

        public void Connect()
        {
            Connect(Timeout.Infinite);
        }

        public void Connect(int timeout)
        {
            ConnectInternal(timeout, CancellationToken.None, Environment.TickCount);
        }

        public Task ConnectAsync()
        {
            // We cannot avoid creating lambda here by using Connect method
            // unless we don't care about start time to be measured before the thread is started
            return ConnectAsync(Timeout.Infinite, CancellationToken.None);
        }

        public Task ConnectAsync(int timeout, CancellationToken cancellationToken)
        {
            int startTime = Environment.TickCount; // We need to measure time here, not in the lambda
            return Task.Run(() => ConnectInternal(timeout, cancellationToken, startTime), cancellationToken);
        }

        private void ConnectInternal(int timeout, CancellationToken cancellationToken, int startTime)
        {
            // 连接的代码
        }
    }
```

通过如上代码可以了解到，实际的连接代码是放在 ConnectInternal 方法里面。在 .NET Framework 下的代码也是差不多的，细节可以忽略

在 ConnectInternal 方法里面，将会进入一个循环，此循环的退出条件只有超时

```csharp
        private void ConnectInternal(int timeout, CancellationToken cancellationToken, int startTime)
        {
            // This is the main connection loop. It will loop until the timeout expires.
            int elapsed = 0;
            SpinWait sw = default;
            do
            {
                cancellationToken.ThrowIfCancellationRequested();

                // Determine how long we should wait in this connection attempt
                int waitTime = timeout - elapsed;
                if (cancellationToken.CanBeCanceled && waitTime > CancellationCheckInterval)
                {
                    waitTime = CancellationCheckInterval;
                }

                // Try to connect.
                if (TryConnect(waitTime, cancellationToken))
                {
                    return;
                }

                // Some platforms may return immediately from TryConnect if the connection could not be made,
                // e.g. WaitNamedPipe on Win32 will return immediately if the pipe hasn't yet been created,
                // and open on Unix will fail if the file isn't yet available.  Rather than just immediately
                // looping around again, do slightly smarter busy waiting.
                sw.SpinOnce();
            }
            while (timeout == Timeout.Infinite || (elapsed = unchecked(Environment.TickCount - startTime)) < timeout);

            throw new TimeoutException();
        }
```

如果调用无参方法，如上面代码，那传入的超时时间是无穷，此时相当于无限循环。在 TryConnect 方法里面，将会尝试连接传入的服务管道名，然而在服务管道没有启动时，是连接不到的，于是 TryConnect 将返回失败。如上面代码，将会进入下一次循环

好在进入循环之前，将会调用 SpinOnce 方法进行自旋。但是无论如何，在连接一个不存在的管道名且没有设置超时时间，将会导致线程进行无限空跑

使用 ConnectAsync 方法时，将使用 `Task.Run` 方法包装，如果此时的连接一个不存在的管道名且没有设置超时时间，将导致当前的线程池的当前执行线程进入无限循环空跑，浪费此线程。而 `Task.Run` 方法将会从线程池调度出一个线程来执行，如果此线程执行了很长时间都没有返回，那么线程池在线程不够用的时候，将会再启动一个新的线程。这就是为什么本文一开始的代码里面，使用 ConnectAsync 方法，将会让 CPU 缓慢上升和线程缓慢上升

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/aacbea5980daa44cabb57d3edde4e1848fe4f8dd/KejeakelcoloqeNemkegudaka) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/aacbea5980daa44cabb57d3edde4e1848fe4f8dd/KejeakelcoloqeNemkegudaka) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin aacbea5980daa44cabb57d3edde4e1848fe4f8dd
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 KejeakelcoloqeNemkegudaka 文件夹
 
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

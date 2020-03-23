# C# dotnet 高性能多线程工具 AsyncAutoResetEvent 异步等待使用方法和原理

在 C# 里面配合 dotnet 的 Task 可以作出 AsyncAutoResetEvent 高性能多线程工具，从命名可以看到 AsyncAutoResetEvent 的意思就是支持异步的自动线程等待事件，用于多线程竞争访问执行权，可以用在消费队列或用在限制有限线程执行的业务上

<!--more-->
<!-- CreateTime:2020/3/21 14:15:17 -->

<!-- 发布 -->

和框架自带的 [AutoResetEvent 类](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.autoresetevent?view=netframework-4.8 ) 一样的作用，表示线程同步事件在一个等待线程释放后收到信号时自动重置

和框架的不同在于 AsyncAutoResetEvent 使用的是异步等待方法，不会在线程池里面阻塞线程，可以让步线程，让线程去处理其他业务

## 适用

作用是支持使用方有多个线程方式访问执行权时，全部都会在 WaitOneAsync 里面阻塞，只有调用 Set 时才会释放，每调用一次释放一个

或者预先多次调用 Set 之后，仅有一个 WaitOneAsync 可以进入

线程在 WaitOneAsync 等待通过 await 而不是阻塞，可以回到线程池执行其他业务，这就是高性能的原因

如有一个线程专门用来制造数据，而有很多个线程需要处理这些数据，此时期望有数据的时候可以均衡自动分配给这些线程处理。没有数据的时候这些线程可以回到线程池里面执行其他业务

如我有并行的业务准备做，但是我期望只使用有限数量的线程去做，虽然我能给的业务量大于可以准备使用的线程数量，但是每次使用的线程数量都小于等于我限制的数量

如我有某个任务需要等待其他任务完成之后才能执行，但是可以等待的任务可以超过多个，也就是多个任务中只要有一个完成了，那么我这个任务就能执行。或者说只要曾经有等待的任务完成过，我的这个任务也能执行

注意，这个库不关注于单个任务的性能，因为会涉及返回线程池等，在 WaitOneAsync 可以进入之后还需要等待线程池调度才能继续，也就是如果 WaitOneAsync 可以进入，但是线程池没有线程可用，那么依然不会执行后续内容。这个库的高性能主要是对整体，通过不阻塞线程的方法最大程度提升性能

这个库开始的设计是用在 WPF 的多个动画播放完成以及对应的事件处理上，虽然本文会说到多线程但不意味真的需要使用多个线程处理。基于 WaitOneAsync 是用 await 会出让的原因，可以通过一个主线程玩出多线程的坑

## 使用方法

通过 NuGet 安装 [dotnetCampus.AsyncWorkerCollection](https://www.nuget.org/packages/dotnetCampus.AsyncWorkerCollection) 库，此项目在 [github](https://github.com/dotnet-campus/AsyncWorkerCollection) 开源

使用 AsyncAutoResetEvent 仅提供两个方法，一个是 WaitOneAsync 另一个是 Set 方法

期望的用法是有很多线程通过 AsyncAutoResetEvent 对象的 WaitOneAsync 进行等待

```csharp
        await asyncAutoResetEvent.WaitOneAsync();
```

然后在另一个线程创建数据或执行某些业务完成之后调用 Set 方法，每调用一次将会让一个在 WaitOneAsync 的线程继续往下执行

```csharp
asyncAutoResetEvent.Set();
```

无论有多少个线程通过 WaitOneAsync 等待，实际上线程都因为使用了 await 而出让执行而不会阻塞，只有等待其他线程调用了 Set 方法，每调用一次将会有一个线程可以继续往下执行

可以使用一个单元测试作为例子

```csharp
                // Arrange
                var asyncAutoResetEvent = new AsyncAutoResetEvent(false);
                var mock = new Mock<IFakeJob>();

                // Action
                asyncAutoResetEvent.Set();
                var task1 = Task.Run(async () =>
                {
                    await asyncAutoResetEvent.WaitOneAsync();
                    mock.Object.Do();
                });

                var task2 = Task.Run(async () =>
                {
                    await asyncAutoResetEvent.WaitOneAsync();
                    mock.Object.Do();
                });

                Task.WaitAny(task1, task2, Task.Delay(TimeSpan.FromSeconds(1)));

                // Assert
                mock.Verify(job => job.Do(), Times.Once);
```

细节如下

- 如果构造函数设置为 true 进入，那么第一个 WaitOneAsync 的线程将会持续执行，第二个 WaitOneAsync 的线程将会等待直到 Set 被调用
- 如果进行 WaitOneAsync 的调用次数小于 Set 调用次数，此时多余的 Set 将之后被记录一次。也就是当不存在任何一个线程在等待 WaitOneAsync 时，再调用 Set 多次，然后如果有多个线程再调用 WaitOneAsync 时，只有一个线程能继续执行，其他线程需要等待 Set 方法调用


## 原理

使用 TaskCompletionSource 支持进行 await 时出让执行，此时的线程会等待 TaskCompletionSource 被调用 SetResult 方法才会继续执行

在调用 WaitOneAsync 的时候，创建一个 TaskCompletionSource 返回给代码用来 await 因此此时 TaskCompletionSource 没有设置 SetResult 方法，也就是代码等待将会出让执行

在调用 Set 方法时才调用其中一个 TaskCompletionSource 的 SetResult 方法让其中一个等待的代码继续执行


```csharp
    public class AsyncAutoResetEvent
    {
        /// <summary>
        /// 提供一个信号初始值，确定是否有信号
        /// </summary>
        /// <param name="initialState">true为有信号，第一个等待可以直接通过</param>
        public AsyncAutoResetEvent(bool initialState)
        {
            _isSignaled = initialState;
        }

        /// <summary>
        /// 异步等待一个信号，需要await
        /// </summary>
        /// <returns></returns>
        public Task WaitOneAsync()
        {
            lock (_locker)
            {
                if (_isSignaled)
                {
                    _isSignaled = false;
                    return CompletedSourceTask;
                }

                var source = new TaskCompletionSource<bool>();
                _waitQueue.Enqueue(source);
                return source.Task;
            }
        }

        /// <summary>
        /// 设置一个信号量，让一个waitone获得信号
        /// </summary>
        public void Set()
        {
            TaskCompletionSource<bool> releaseSource = null;
            lock (_locker)
            {
                if (_waitQueue.Any())
                {
                    releaseSource = _waitQueue.Dequeue();
                }

                if (releaseSource is null)
                {
                    if (!_isSignaled)
                    {
                        _isSignaled = true;
                    }
                }
            }

            releaseSource?.SetResult(true);
        }

        private static readonly Task CompletedSourceTask = Task.FromResult(true);

        private readonly Queue<TaskCompletionSource<bool>> _waitQueue =
            new Queue<TaskCompletionSource<bool>>();

        private bool _isSignaled;

        private readonly object _locker = new object();
    }
```

源代码请看 [https://github.com/dotnet-campus/AsyncWorkerCollection/blob/480ba1159289eebf0e08996f866a4fa832099f4b/AsyncWorkerCollection/AsyncAutoResetEvent.cs](https://github.com/dotnet-campus/AsyncWorkerCollection/blob/480ba1159289eebf0e08996f866a4fa832099f4b/AsyncWorkerCollection/AsyncAutoResetEvent.cs)

## 测试

此库其实在我的几个项目里面经过了一年的测试，大概在100w台设备上运行过，没有翻车

这样的库实际上单元测试作用不大……

## 感谢

此库 [dotnet-campus/AsyncWorkerCollection: 多线程异步工具](https://github.com/dotnet-campus/AsyncWorkerCollection ) 由多线程砖家[头像](https://xinyuehtx.github.io/ ) 用了一年的时间写的，因为自己业务使用也许没有测试出坑，于是开源出来，请小伙伴协助测试

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

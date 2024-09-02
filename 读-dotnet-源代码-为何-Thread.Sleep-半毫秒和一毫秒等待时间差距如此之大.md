
# 读 dotnet 源代码 为何 Thread.Sleep 半毫秒和一毫秒等待时间差距如此之大

本文记录我读 dotnet 的源代码了解到为什么调用 Thread.Sleep 的时候，传入的是不足一毫秒，如半毫秒时或 0.99 毫秒，与传入是一毫秒时，两者的等待时间差距非常大

<!--more-->


<!-- CreateTime:2024/08/27 07:12:02 -->

<!-- 发布 -->
<!-- 博客 -->

大概如下的代码，分别进行两次传入给 Thread.Sleep 不同等待时间的循环测试。其中一次传入的是 0.99 毫秒，一次传入的是 1 毫秒

```csharp
using System.Diagnostics;

var stopwatch = Stopwatch.StartNew();

for (int i = 0; i < 1000; i++)
{
    Thread.Sleep(TimeSpan.FromMilliseconds(0.99));
}

stopwatch.Stop();

Console.WriteLine($"耗时：{stopwatch.ElapsedMilliseconds}ms");

stopwatch.Restart();
for (int i = 0; i < 1000; i++)
{
    Thread.Sleep(TimeSpan.FromMilliseconds(1));
}
Console.WriteLine($"耗时：{stopwatch.ElapsedMilliseconds}ms");
```

在我的设备上运行的输出内容如下

```
耗时：0ms
耗时：15665ms
```

通过如上代码可以看到传入 0.99 毫秒时，居然接近统计不出来其耗时

而传入 1 毫秒时，由于在 Windows 下的最低 Thread.Sleep 时间大概在 15-16毫秒 左右，于是差不多是 15 秒左右的时间，这是符合预期的。即写入 `Thread.Sleep(TimeSpan.FromMilliseconds(1));` 也可能差不多等待 15 毫秒的量程时间

那为什么 0.99 毫秒和 1 毫秒只差大概 0.1 毫秒的时间，却在等待过程中有如此长的时间差距

通过阅读 dotnet 的源代码，可以看到 Thread.Sleep 的实现代码大概如下

```csharp
namespace System.Threading
{
    public sealed partial class Thread
    {
        ... // 忽略其他代码
        public static void Sleep(TimeSpan timeout) => Sleep(WaitHandle.ToTimeoutMilliseconds(timeout));
    }
}

namespace System.Threading
{
    public abstract partial class WaitHandle : MarshalByRefObject, IDisposable
    {
        internal static int ToTimeoutMilliseconds(TimeSpan timeout)
        {
            long timeoutMilliseconds = (long)timeout.TotalMilliseconds;
             ... // 忽略其他代码
            return (int)timeoutMilliseconds;
        }
        ... // 忽略其他代码
    }
}
```

通过以上可以可见，这是直接将 TotalMilliseconds 强行转换为 int 类型，换句话说就是不到 1 毫秒的，都会被转换为 0 毫秒的值

于是即使是 0.99 毫秒，在这里的转换之下，依然会返回 0 毫秒回去

而 Thread.Sleep 底层里面专门为传入 0 毫秒做了特殊处理，~~将会进入自旋逻辑。大家都知道，进入自旋时，自旋的速度是非常快的~~ 将会直接出让线程执行时间片。也就是说假设系统给当前线程分配了 10 毫秒的执行时间，当前线程执行到 Thread.Sleep 之前，只花了 5 毫秒，当执行了 Thread.Sleep 将会直接让线程出让执行权，无论线程还剩余多少可执行时间。出让之后线程会重新加入调度，这个过程也是非常快速的。在 dotnet 里面的自旋 SpinWait 辅助类里面，是对 Thread.Sleep 和 Thread.Yield 之间的封装，确保不会进入长时间的自旋而导致影响系统整体运行

从狭义的自旋锁定义上看，自旋锁要求线程在这一过程中保持执行。因此自旋锁从定义上和 Thread.Sleep 会出让的行为是冲突的。但是在工程上，实现“自旋”概念的行为时，却会间断采用 `Thread.Sleep(0)` 等出让的方式，用于减少 CPU 的空转，如以下的 dotnet 源代码所示

```csharp
        public void SpinOnce(int sleep1Threshold)
        {
            ArgumentOutOfRangeException.ThrowIfLessThan(sleep1Threshold, -1);

            if (sleep1Threshold >= 0 && sleep1Threshold < YieldThreshold)
            {
                sleep1Threshold = YieldThreshold;
            }

            SpinOnceCore(sleep1Threshold);
        }

        private void SpinOnceCore(int sleep1Threshold)
        {
            Debug.Assert(sleep1Threshold >= -1);
            Debug.Assert(sleep1Threshold < 0 || sleep1Threshold >= YieldThreshold);

            // (_count - YieldThreshold) % 2 == 0: The purpose of this check is to interleave Thread.Yield/Sleep(0) with
            // Thread.SpinWait. Otherwise, the following issues occur:
            //   - When there are no threads to switch to, Yield and Sleep(0) become no-op and it turns the spin loop into a
            //     busy-spin that may quickly reach the max spin count and cause the thread to enter a wait state, or may
            //     just busy-spin for longer than desired before a Sleep(1). Completing the spin loop too early can cause
            //     excessive context switcing if a wait follows, and entering the Sleep(1) stage too early can cause
            //     excessive delays.
            //   - If there are multiple threads doing Yield and Sleep(0) (typically from the same spin loop due to
            //     contention), they may switch between one another, delaying work that can make progress.
            if ((
                    _count >= YieldThreshold &&
                    ((_count >= sleep1Threshold && sleep1Threshold >= 0) || (_count - YieldThreshold) % 2 == 0)
                ) ||
                Environment.IsSingleProcessor)
            {
                //
                // We must yield.
                //
                // We prefer to call Thread.Yield first, triggering a SwitchToThread. This
                // unfortunately doesn't consider all runnable threads on all OS SKUs. In
                // some cases, it may only consult the runnable threads whose ideal processor
                // is the one currently executing code. Thus we occasionally issue a call to
                // Sleep(0), which considers all runnable threads at equal priority. Even this
                // is insufficient since we may be spin waiting for lower priority threads to
                // execute; we therefore must call Sleep(1) once in a while too, which considers
                // all runnable threads, regardless of ideal processor and priority, but may
                // remove the thread from the scheduler's queue for 10+ms, if the system is
                // configured to use the (default) coarse-grained system timer.
                //

                if (_count >= sleep1Threshold && sleep1Threshold >= 0)
                {
                    Thread.Sleep(1);
                }
                else
                {
                    int yieldsSoFar = _count >= YieldThreshold ? (_count - YieldThreshold) / 2 : _count;
                    if ((yieldsSoFar % Sleep0EveryHowManyYields) == (Sleep0EveryHowManyYields - 1))
                    {
                        Thread.Sleep(0);
                    }
                    else
                    {
                        Thread.Yield();
                    }
                }
            }
            else
            {
                //
                // Otherwise, we will spin.
                //
                // We do this using the CLR's SpinWait API, which is just a busy loop that
                // issues YIELD/PAUSE instructions to ensure multi-threaded CPUs can react
                // intelligently to avoid starving. (These are NOOPs on other CPUs.) We
                // choose a number for the loop iteration count such that each successive
                // call spins for longer, to reduce cache contention.  We cap the total
                // number of spins we are willing to tolerate to reduce delay to the caller,
                // since we expect most callers will eventually block anyway.
                //
                // Also, cap the maximum spin count to a value such that many thousands of CPU cycles would not be wasted doing
                // the equivalent of YieldProcessor(), as at that point SwitchToThread/Sleep(0) are more likely to be able to
                // allow other useful work to run. Long YieldProcessor() loops can help to reduce contention, but Sleep(1) is
                // usually better for that.
                int n = Thread.OptimalMaxSpinWaitsPerSpinIteration;
                if (_count <= 30 && (1 << _count) < n)
                {
                    n = 1 << _count;
                }
                Thread.SpinWait(n);
            }

            // Finally, increment our spin counter.
            _count = (_count == int.MaxValue ? YieldThreshold : _count + 1);
        }
```

以上的代码虽然是写在 SpinOnce 里面，但不意味着一定会占用着 CPU 进行空跑，而是会根据其等待时间决定是否将线程切出去，从而最大化利用系统资源

通过上文的分析，可以看到 `Thread.Sleep(TimeSpan.FromMilliseconds(0.99));` 代码和 `Thread.Sleep(0)` 在执行上等价的，意味着第一次只执行了一千次~~自旋~~线程出让，自然就几乎测试不出来耗时了

在 Windows 下的 Thread.Sleep 底层代码是写在 Thread.Windows.cs 代码里的，实现如下

```csharp
namespace System.Threading
{
    public sealed partial class Thread
    {
        internal static void UninterruptibleSleep0() => Interop.Kernel32.Sleep(0);

#if !CORECLR
        private static void SleepInternal(int millisecondsTimeout)
        {
            Debug.Assert(millisecondsTimeout >= -1);
            Interop.Kernel32.Sleep((uint)millisecondsTimeout);
        }
#endif

        ... // 忽略其他代码
    }
}
```

如上面代码，底层为 Kernel32 的 Sleep 函数，如[官方文档](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep)所述，传入 0 是特殊的实现逻辑

> If you specify 0 milliseconds, the thread will relinquish the remainder of its time slice but remain ready.

因此如果在 Thread.Sleep 方法里面传入的 TimeSpan 不足一毫秒，那就和传入 0 毫秒是相同的执行逻辑

更多基础技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
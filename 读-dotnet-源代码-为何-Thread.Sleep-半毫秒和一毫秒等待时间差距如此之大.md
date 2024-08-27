
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

而 Thread.Sleep 底层里面专门为传入 0 毫秒做了特殊处理，将会进入自旋逻辑。大家都知道，进入自旋时，自旋的速度是非常快的

以上的 `Thread.Sleep(TimeSpan.FromMilliseconds(0.99));` 代码和 `Thread.Sleep(0)` 在执行上等价的，意味着第一次只执行了一千次自旋，自然就几乎测试不出来耗时了

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
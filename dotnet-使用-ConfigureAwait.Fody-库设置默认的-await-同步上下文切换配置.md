
# dotnet 使用 ConfigureAwait.Fody 库设置默认的 await 同步上下文切换配置

在 dotnet 里面，使用 await 进行异步逻辑，默认是会尝试切换回调用 await 的线程同步上下文。这个机制对于大多数的上层应用来说都是符合逻辑且方便的逻辑，例如对于带 UI 线程的 WPF 或 WinForms 等应用，基础开发的执行逻辑基本都是在 UI 线程上，此时进入一次 await 再出来，期望如果是进入 await 之前是在 UI 线程，那么执行 await 完成之后，退出的代码也能在 UI 线程执行，正好这就是 dotnet 的默认行为。但是对于库开发者来说，情况就反过来的，库的开发者大部分时候更期望默认不要切换回调用方的线程，采用 Fody 的 ConfigureAwait.Fody 库，可以控制此默认的行为。本文将告诉大家如何使用 ConfigureAwait.Fody 库

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

这是一个在 GitHub 上使用最友好的 MIT 协议开源的库，请看 [https://github.com/Fody/ConfigureAwait](https://github.com/Fody/ConfigureAwait)

用 Fody 的 ConfigureAwait.Fody 库，可以控制 await 在结束之后的切换同步上下文默认的行为，对于库的开发者来说相对比较方便。大部分的库的逻辑，都是期望在异步之后，不要明确切换回原调用方的线程，因为切换回原调用方的线程存在很多不可控逻辑。例如在 WPF 里面，需要通过 Dispatcher 调度，如此会让 UI 线程过于繁忙。而且切换调度逻辑，可能出现和原有线程相互等待的情况

例如 UI 线程进入了 Wait 逻辑，等待异步执行完成。然而异步执行完成的最后一步是做切换线程同步上下文，切换到 UI 线程。大家可以看到，异步的最后一步是在等待 UI 线程切换，相当于在 WPF 里面使用 Dispatcher 调度，然而 UI 线程却进入了 Wait 方法，也就是 UI 线程在异步完成之后无法进行调度。此时的异步在等待 UI 调度，而 UI 在等待异步完成。如此将会锁住 UI 线程

详细请看 

- [将 async/await 异步代码转换为安全的不会死锁的同步代码（使用 PushFrame） - walterlv](https://blog.walterlv.com/post/convert-async-to-sync-by-push-frame.html )

- [使用 Task.Wait()？立刻死锁（deadlock） - walterlv](https://blog.walterlv.com/post/deadlock-in-task-wait.html )

- [在编写异步方法时，使用 ConfigureAwait(false) 避免使用者死锁 - walterlv](https://blog.walterlv.com/post/using-configure-await-to-avoid-deadlocks.html )

根据 walterlv 大佬的 [在编写异步方法时，使用 ConfigureAwait(false) 避免使用者死锁 - walterlv](https://blog.walterlv.com/post/using-configure-await-to-avoid-deadlocks.html ) 博客，可以了解到，在库里面，如果不关心线程本身，例如代码不需要在 WPF 的 UI 线程执行，可以采用 ConfigureAwait(false) 的方式避免使用者死锁

原因在于在 await 完成前，可以采用 ConfigureAwait 配置异步的最后一步是否需要尝试切换回原有的线程。默认是 true 的值，表示需要。如果加上了 ConfigureAwait 函数，设置 false 的值，那就表示不要切换回原有的线程。此时如果业务端在 UI 线程使用 Wait 等方法，那依然是安全的，原因是 UI 线程在等待异步完成，然而异步完成不需要调度回 UI 线程，可以由线程池选择线程调度，于是异步的完成不需要等待 UI 线程，能够让 UI 线程等待异步完成

那引入的问题就来了，在库里面，将会让 await 异步逻辑充满了 ConfigureAwait(false) 的代码，如此将会让代码不好看。用 Fody 的 ConfigureAwait.Fody 库就是用来解决此问题的，可以配置默认行为，例如 dotnet 里面默认是用的是 true 的值，对于库的代码，可以反过来，配置默认是 false 的值，可以减少大量的代码

按照 dotnet 的使用惯例，第一步就是先安装 NuGet 库。由于 ConfigureAwait.Fody 库是 Fody 库的扩展，请同时安装 [ConfigureAwait.Fody](https://nuget.org/packages/ConfigureAwait.Fody/) 和 [Fody](https://nuget.org/packages/Fody/) 库

使用方法很灵活，可以配置整个程序集的默认行为，也可以只配置某个类或类里面某个方法的默认行为。配置整个程序集的默认行为代码如下，添加程序集的特性即可

```csharp
[assembly: Fody.ConfigureAwait(false)]
```

对于某个类或类里面某个方法的默认行为的配置，可以给类或方法加上如下特性

```csharp
[Fody.ConfigureAwait(false)]
```

例子如下

```csharp
        [Fody.ConfigureAwait(false)]
        private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            Debug.WriteLine($"ThreadId={Thread.CurrentThread.ManagedThreadId}"); // 输出 1
            await Task.Delay(100);
            Debug.WriteLine($"ThreadId={Thread.CurrentThread.ManagedThreadId}"); // 输出 2
        }
```

如此即可配置行为为加上 ConfigureAwait(false) 不尝试切换回原因的线程同步上下文

按照 Fody 的使用方法，加上 FodyWeavers.xml 文件，在 FodyWeavers.xml 文件里面开启 ConfigureAwait.Fody 的功能

```xml
<Weavers>
  <ConfigureAwait/>
</Weavers>
```

如果想对程序集做默认配置，也可以不写程序集特性，可以通过在 FodyWeavers.xml 文件里设置默认值的方式实现

```xml
<Weavers>
  <ConfigureAwait ContinueOnCapturedContext="false" />
</Weavers>
```

实现原理是编译器优化，如原本的代码如下

```csharp
using Fody;

[ConfigureAwait(false)]
public class MyAsyncLibrary
{
    public async Task MyMethodAsync()
    {
        await Task.Delay(10);
        await Task.Delay(20);
    }

    public async Task AnotherMethodAsync()
    {
        await Task.Delay(30);
    }
}
```

将会编译生成大概如下等价代码

```csharp
public class MyAsyncLibrary
{
    public async Task MyMethodAsync()
    {
        await Task.Delay(10).ConfigureAwait(false);
        await Task.Delay(20).ConfigureAwait(false);
    }

    public async Task AnotherMethodAsync()
    {
        await Task.Delay(30).ConfigureAwait(false);
    }
}
```

相当于不需要开发者手动加上 ConfigureAwait 方法，通过此工具自动加上





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
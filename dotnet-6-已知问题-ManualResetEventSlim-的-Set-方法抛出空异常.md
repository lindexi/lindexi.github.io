
# dotnet 6 已知问题 ManualResetEventSlim 的 Set 方法抛出空异常

本文记录一个 dotnet 6 已知问题，此问题预计是在 .NET Framework 4.5 时就引入的，我没有考古在 .NET Framework 4.5 之前是否还存在此问题。当前这个问题在 .NET 7 修复

<!--more-->


<!-- CreateTime:2023/6/25 8:43:13 -->

<!-- 发布 -->
<!-- 博客 -->

这是从我的埋点上报遥测收集到的异常信息，在线程池里面的调用堆栈，调用到 ManualResetEventSlim 的 Set 方法，抛出了 System.NullReferenceException 异常，堆栈如下

```
System.NullReferenceException: Object reference not set to an instance of an object.
   in void ManualResetEventSlim.Set(bool duringCancellation)
   in void Task.FinishStageTwo()
   in void Task.FinishSlow(bool userDelegateExecute)
   in bool ThreadPoolWorkQueue.Dispatch()
   in void WorkerThread.WorkerThreadStart()
   in void Thread.StartCallback()
```

我采用的 dotnet 框架是 `6.0.13` 版本

这个异常发生的次数非常少，在大概一千万的用户里面只有三个用户发送过这个问题

我将这个问题报告给官方： [https://github.com/dotnet/runtime/issues/87761](https://github.com/dotnet/runtime/issues/87761)

我预计这个问题属于多线程安全问题，而且通过异常的调用堆栈可以看到里面没有我编写的业务代码，大概可以证明是底层 dotnet 框架的问题

通过以上堆栈的 `ThreadPoolWorkQueue.Dispatch` 大概可以了解到属于线程池模块，在这里如果抛出了异常，属于线程顶层异常，应用程序进程是接不住的，将会闪退

换句话说就是遇到这个异常，约等于进程将会被炸掉

由于异常发生的次数太少，我也没有调查出来具体原因，而且进一步阅读 dotnet 仓库的源代码，我也没有找到任何可能在 Set 方法里面抛出的空异常

大佬回复我说这个问题预计是被在 .NET 7 的清理旧代码时，顺手修掉了，修复的代码请看 [https://github.com/dotnet/runtime/pull/71779/files#diff-f190bff628bded0860cc435bb5fc7d0e4b85d23aefbdae14e2f72986a0e295daR316](https://github.com/dotnet/runtime/pull/71779/files#diff-f190bff628bded0860cc435bb5fc7d0e4b85d23aefbdae14e2f72986a0e295daR316 )

```csharp
-           if (m_eventObj != null)
-           {
-               m_eventObj.Reset();
-           }
+           m_eventObj?.Reset();
```

核心问题就是之前的 ManualResetEventSlim 存放的静态字段 `m_eventObj` 可能被在多线程执行时，在 `if (m_eventObj != null)` 判断非空时通过，然而在 `m_eventObj.Reset();` 使用就被赋值为空

更新代码使用新语法加上问号即可修复此问题。加上问号之后，将会先捕获 `m_eventObj` 对象作为一个变量，接着判断变量是否为空，不空才执行 Reset 方法，等同于以下代码

```csharp
var eventObj = m_eventObj;

if (eventObj != null)
{
    eventObj.Reset();
}
```

由于捕获了局部变量，从而规避了多线程赋空值安全问题

由于我阅读 dotnet 代码的时候看的是 main 分支的代码，这部分和 dotnet 6 的有差别，从而没有能够找到问题

这里也再次感谢 [Stephen Toub](https://github.com/stephentoub) 大佬




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
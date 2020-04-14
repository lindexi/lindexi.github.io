# WPF 升级 .NET Core 的理由

本文列举一些让 WPF 升级 .NET Core 的理由

<!--more-->
<!-- CreateTime:4/10/2020 11:12:19 AM -->

<!-- 发布 -->

## 提供更多的 API 同时提升运行性能

为了支持 Win7 系统，限制了 .NET Framework 升级版本

当前我团队的 .NET Framework 使用 4.5 但是如果使用 dotnet core 能使用更多的 API 同时这些 API 都优化了大部分性能

## 启动性能优化

在 dotnet core 2.2 提供的阶梯编译，可以提升启动过程的 JIT 编译速度

## 环境问题

可以全添加所有依赖的包，可以解决 .NET Framework 环境问题

### 修复 D3D Compile47 问题

[Adding d3d_compiler dependency to known issues by rladuca · Pull Request #190 · dotnet/wpf](https://github.com/dotnet/wpf/pull/190 )

[WPF Applications require crash with System.TypeLoadException when VC++ redistributables are not present · Issue #37 · dotnet/wpf](https://github.com/dotnet/wpf/issues/37 )

更多关于 D3D Compile47 问题，请看 [win7 无法启动 WPF 程序 D3Dcompiler_47.dll 丢失](https://blog.lindexi.com/post/win7-%E6%97%A0%E6%B3%95%E5%90%AF%E5%8A%A8-WPF-%E7%A8%8B%E5%BA%8F-D3Dcompiler_47.dll-%E4%B8%A2%E5%A4%B1.html)

## 触摸问题修复

### 修复特定硬件带崩软件

修复特定硬件带崩软件，需要在 .NET 4.7.1 和 Win10 系统才能生效

[dotnet/481090-WPF Touch generates NullReferenceException in ProcessInputReport.md at master · Microsoft/dotnet](https://github.com/Microsoft/dotnet/blob/master/releases/net471/KnownIssues/481090-WPF%20Touch%20generates%20NullReferenceException%20in%20ProcessInputReport.md )

This issue is fixed for all supported OS platforms prior to Windows 10 Fall Creators Update. The fix for Windows 10 Fall Creators Update is expected in a future servicing update.


```csharp
 System.Windows.Input.StylusWisp.WispLogic.ProcessInputReport(RawStylusInputReport inputReport) 
 System.Windows.Input.PenContext.FirePackets(Int32 stylusPointerId, Int32[] data, Int32 timestamp) 
 System.Windows.Input.PenThreadWorker.FlushCache(Boolean goingOutOfRange) 
 System.Windows.Input.PenThreadWorker.ThreadProc() 
 System.Threading.ThreadHelper.ThreadStart_Context(Object state) 
 System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx) 
 System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx) 
 System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state) 
 System.Threading.ThreadHelper.ThreadStart()

```

### 书写索引超出了数组界限

此问题已经报告微软 [Throw IndexOutOfRangeException in WispLogic.CoalesceAndQueueStylusEvent · Issue #935 · dotnet/wpf](https://github.com/dotnet/wpf/issues/935 )

```
在 System.Collections.Generic.Dictionary`2.Insert(TKey key, TValue value, Boolean add)
在 System.Windows.Input.StylusWisp.WispLogic.CoalesceAndQueueStylusEvent(RawStylusInputReport inputReport)
在 System.Windows.Input.StylusWisp.WispLogic.ProcessInputReport(RawStylusInputReport inputReport)
在 System.Windows.Input.PenContext.FirePackets(Int32 stylusPointerId, Int32[] data, Int32 timestamp)
在 System.Windows.Input.PenThreadWorker.FlushCache(Boolean goingOutOfRange)
在 System.Windows.Input.PenThreadWorker.FireEvent(PenContext penContext, Int32 evt, Int32 stylusPointerId, Int32 cPackets, Int32 cbPacket, IntPtr pPackets)
在 System.Windows.Input.PenThreadWorker.ThreadProc()
在 System.Threading.ThreadHelper.ThreadStart_Context(Object state)
在 System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
在 System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
在 System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)
在 System.Threading.ThreadHelper.ThreadStart()

ExceptionType: System.IndexOutOfRangeException
ExceptionMessage: 索引超出了数组界限

```



### 触摸事件

在 .NET Core 和 .NET Framework 4.8 修复了在 StylusUp 抛异常等让下次触摸失效


## Popup 修复

### Popup 触摸问题

修复 Popup 触摸失效，需要在 .NET 4.7.1 和 Win10 系统才能生效

[dotnet/479874-WPF Touch Stops Working After Prolonged Use of Popups.md at master · Microsoft/dotnet](https://github.com/Microsoft/dotnet/blob/master/releases/net471/KnownIssues/479874-WPF%20Touch%20Stops%20Working%20After%20Prolonged%20Use%20of%20Popups.md ) 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

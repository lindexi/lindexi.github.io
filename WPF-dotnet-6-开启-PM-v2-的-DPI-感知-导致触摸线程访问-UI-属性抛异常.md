
# WPF dotnet 6 开启 PM v2 的 DPI 感知 导致触摸线程访问 UI 属性抛异常

本文记录一个 WPF 在 dotnet 6 的一个已知问题，且此问题我已修复提交给官方仓库。这是一个只有在 dotnet 6 框架下，非 dotnet 5 也非 .NET Core 3.1 也非 .NET Framework 的问题，要求开启 DPI 感觉等级为 PerMonitorV2 的特性，在带触摸屏上的应用，应用运行过程中，切换屏幕的 DPI 之后，触摸过程有概率触发在触摸线程访问 UI 的依赖属性，在触摸线程抛出异常炸掉应用

<!--more-->


<!-- CreateTime:2022/7/25 8:28:00 -->

<!-- 发布 -->
<!-- 博客 -->

## 条件

必须同时满足以下条件：

- dotnet 6: dotnet 6.0.1 及以上版本
  - dotnet 5 和 .NET Core 3.1 和 .NET Framework 没有此问题，这是新改出来的，细节请参阅原理部分
- 应用开启 PerMonitorV2 的特性
  - 支持此特性最低系统版本是 Windows 10 的 1703 版本，低于此版本，包括 Win7 系统，将不能开启
  - 默认的应用是没有开启的，需要自己通过清单等方式开启，开启方法稍微复杂，请参阅 [支持 Windows 10 最新 PerMonitorV2 特性的 WPF 多屏高 DPI 应用开发 - walterlv](https://blog.walterlv.com/post/windows-high-dpi-development-for-wpf.html )
- 应用开启 StylusPlugIn 的支持  
- 在触摸设备上运行，进行触摸交互
- 应用运行过程存在切换系统的 DPI 的值
  - 需要先运行应用，对应用进行触摸交互，再切换，再触摸
  - 可以选择多个屏幕不同的 DPI 让 WPF 在多个屏幕来回移动和触摸
  - 可以选择一个屏幕，在运行应用过程切换 DPI 的值

这也算是一个好消息，要求很严格，而且在用户端，很多都是只有一个屏幕。再加上切换 DPI 系统会提示要重启电脑，重启电脑就不会存在此问题。也就是说这个问题影响其实是比较小的

最后也是最重要的是，这个 Bug 不是必复现的，也许你需要很多次测试才可以遇到，详细请参阅下面步骤

## 步骤

如以上条件，在 Win10 的 1703 以上版本运行，通过 [支持 Windows 10 最新 PerMonitorV2 特性的 WPF 多屏高 DPI 应用开发 - walterlv](https://blog.walterlv.com/post/windows-high-dpi-development-for-wpf.html ) 博客的方法给应用开启 PM v2 的功能

根据以上条件，给应用附加上 StylusPlugIn 的支持，方法请参阅 [附加 StylusPlugIn 的例子](https://blog.lindexi.com/post/WPF-%E5%A4%9A%E4%B8%AA-StylusPlugIn-%E7%9A%84%E4%BA%8B%E4%BB%B6%E8%A7%A6%E5%8F%91%E9%A1%BA%E5%BA%8F.html )

准备完成之后，执行以下步骤

1. 启动应用，进行触摸

2. 接着打开设置，点击屏幕选项卡，修改缩放和布局的 更改文本、应用等项目的大小，修改百分比

3. 切换回应用，继续触摸应用

这是一个非必定复现的坑，需要多次循环以上步骤，也许才能遇到此坑。行为是在触摸线程 Stylus Input 线程将会因为调用的 GetAndCacheTransformToDeviceMatrix 方法碰了 UI 线程的属性，抛出如下异常

```
Application: Application.exe
CoreCLR Version: 6.0.121.56705
.NET Version: 6.0.1
Description: The process was terminated due to an unhandled exception.
Exception Info: System.InvalidOperationException: The calling thread cannot access this object because a different thread owns it.
   at System.Windows.Threading.Dispatcher.ThrowVerifyAccess()
   at System.Windows.Threading.Dispatcher.VerifyAccess()
   at System.Windows.Threading.DispatcherObject.VerifyAccess()
   at System.Windows.Media.CompositionTarget.VerifyAPIReadOnly()
   at System.Windows.Interop.HwndTarget.get_TransformToDevice()
   at System.Windows.Input.StylusLogic.GetAndCacheTransformToDeviceMatrix(PresentationSource source)
   at System.Windows.Input.StylusWisp.WispLogic.GetTabletToViewTransform(PresentationSource source, TabletDevice tabletDevice)
   at System.Windows.Input.PenContexts.InvokeStylusPluginCollection(RawStylusInputReport inputReport)
   at System.Windows.Input.StylusWisp.WispLogic.InvokeStylusPluginCollection(RawStylusInputReport inputReport)
   at System.Windows.Input.StylusWisp.WispLogic.ProcessInputReport(RawStylusInputReport inputReport)
   at System.Windows.Input.StylusWisp.WispLogic.ProcessInput(RawStylusActions actions, PenContext penContext, Int32 tabletDeviceId, Int32 stylusDeviceId, Int32[] data, Int32 timestamp, PresentationSource inputSource)
   at System.Windows.Input.PenContexts.ProcessInput(RawStylusActions actions, PenContext penContext, Int32 tabletDeviceId, Int32 stylusPointerId, Int32[] data, Int32 timestamp)
   at System.Windows.Input.PenContexts.OnPenDown(PenContext penContext, Int32 tabletDeviceId, Int32 stylusPointerId, Int32[] data, Int32 timestamp)
   at System.Windows.Input.PenContext.FirePenDown(Int32 stylusPointerId, Int32[] data, Int32 timestamp)
   at System.Windows.Input.PenThreadWorker.FireEvent(PenContext penContext, Int32 evt, Int32 stylusPointerId, Int32 cPackets, Int32 cbPacket, IntPtr pPackets)
   at System.Windows.Input.PenThreadWorker.ThreadProc()
   at System.Threading.Thread.StartHelper.Callback(Object state)
   at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state)
--- End of stack trace from previous location ---
   at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state)
   at System.Threading.Thread.StartCallback()
```

如果自己试了几次也没有复现，可以试试用我的版本，保证按照上面步骤，一定挂。我的版本由以下三个 NuGet 包组成

- [https://www.nuget.org/packages/dotnetCampus.WPF/6.0.4-alpha05-FixTouch01](https://www.nuget.org/packages/dotnetCampus.WPF/6.0.4-alpha05-FixTouch01)
- [https://www.nuget.org/packages/dotnetCampus.WPF.Resource/6.0.4-alpha05-FixTouch01](https://www.nuget.org/packages/dotnetCampus.WPF.Resource/6.0.4-alpha05-FixTouch01)
- [https://www.nuget.org/packages/dotnetCampus.WPF.Dependencies/6.0.4-alpha05-FixTouch01](https://www.nuget.org/packages/dotnetCampus.WPF.Dependencies/6.0.4-alpha05-FixTouch01)

相信想用定制版本的 WPF 的开发者都知道可以使用吧

为什么使用 6.0.4-alpha05-FixTouch01 版本是能一定复现，还请看下面的原理部分

## 原理

为什么使用 6.0.4-alpha05-FixTouch01 版本是能一定复现，那是因为我改了触摸模块，我修复了触摸偏移问题导致了此问题暴露。为什么有触摸问题？这是因为 [Rob LaDuca](https://github.com/rladuca) 大佬在 [Fix raw stylus data to support per-monitor DPI by rladuca · Pull Request #2891 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2891) 修复了 PM 的触摸问题，然而他的修复引入新的问题。我问他，你有触摸屏测试没，他说没有，不过 WPF 内部有个自动化测试，自动化测试通过就可以了。然而他的更改已合入主干，导致了使用 StylusPlugIn 的触摸存在偏移

我在 [Try fix the first point in StylusPlugin in high DPI by lindexi · Pull Request #6428 · dotnet/wpf](https://github.com/dotnet/wpf/pull/6428) 修复了以上的触摸偏移问题，但是由于此修复引入了新的问题。修复之前，如 [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html ) 描述，将会在 UI 线程收到触摸之前，先在触摸线程收到。在触摸线程收到时，还没有找到命中的元素，这就导致了拿到的空值，无法处理当前命中到的元素所在的窗口，从而无法了解当前触摸点的 DPI 的参数。于是触摸就因为拿不到 DPI 参数进行计算而偏移

我修复了触摸偏移问题是通过拿触摸输入源的窗口句柄进行获取 DPI 计算。获取触摸的输入源窗口，不需要等待 UI 线程命中测试，于是修复了触摸偏移的问题

然而以上输入引入了新的问题，那就是在开启 PM v2 特性，在 DPI 变更之后，触摸比 UI 线程更快进入 GetAndCacheTransformToDeviceMatrix 方法。 此方法的作用是获取或计算 DPI 换算 Matrix 参数。如果是在 UI 线程先进来，那自然能更新为一个符合预期的值。然而如果是触摸线程先进来，将会由于触摸线程没有从 `_transformToDeviceMatrices` 字典获取到对应的 DPI 的参数，从而需要获取 TransformToDevice 属性。在获取 TransformToDevice 属性的时候，由于 TransformToDevice 属性默认是限制只有 UI 线程可以访问，于是就抛出了异常

以下是 GetAndCacheTransformToDeviceMatrix 代码，我添加了足够的注释，方便大家了解

```csharp
 protected Matrix GetAndCacheTransformToDeviceMatrix(PresentationSource source) 
 { 
 	 // 在当前 dotnet 主干分支上，由于 Rob LaDuca 大佬修复 per-monitor DPI 时，没有考虑到 StylusPlugIn 比 UI 线程更快进入此函数，在首次触摸时，让 PresentationSource 参数为空，从而无法获取到正确的值进行计算，从而计算触摸点由于缺少参数，在 DPI 非 96 情况下偏移 DPI 比例

     var hwndSource = source as HwndSource; 
     Matrix toDevice = Matrix.Identity; 
  
     if (hwndSource?.CompositionTarget != null) 
     {
     	 // 如果更改了 DPI 且开启特性，那么在触摸线程比 UI 线程更快进入此函数时，将会在 _transformToDeviceMatrices 字典里面获取不到参数，需要 触摸线程 计算
         // If we have not yet seen this DPI, store the matrix for it. 
         if (!_transformToDeviceMatrices.ContainsKey(hwndSource.CompositionTarget.CurrentDpiScale)) 
         { 
         	 // 触摸线程获取 TransformToDevice 参数，将会因为 TransformToDevice 参数默认限制只有 UI 线程可以访问从而炸掉
             _transformToDeviceMatrices[hwndSource.CompositionTarget.CurrentDpiScale] = hwndSource.CompositionTarget.TransformToDevice; 
             Debug.Assert(_transformToDeviceMatrices[hwndSource.CompositionTarget.CurrentDpiScale].HasInverse); 
         } 
  
         toDevice = _transformToDeviceMatrices[hwndSource.CompositionTarget.CurrentDpiScale]; 
     } 
  
     return toDevice; 
 } 
```

问题已反馈给 WPF 官方： [WPF tocuh in Window with StylusPlugIn may throw InvalidOperationException · Issue #6829 · dotnet/wpf](https://github.com/dotnet/wpf/issues/6829)

在 [少珺](https://github.com/kkwpsv) 小伙伴的帮助下，我修复了此问题，请看 [Fix get TransformToDevice in Stylus Input thread will throw the InvalidOperationException by lindexi · Pull Request #6840 · dotnet/wpf](https://github.com/dotnet/wpf/pull/6840)

核心修复的方法是在触摸线程计算，而不是获取 TransformToDevice 属性，这是因为 TransformToDevice 属性的获取方法里面也是一个简单的计算。从性能角度和安全角度都是自己计算会更好

```csharp
 public override Matrix TransformToDevice 
 { 
     get 
     { 
         VerifyAPIReadOnly(); 
         Matrix m = Matrix.Identity; 
         m.Scale(CurrentDpiScale.DpiScaleX, CurrentDpiScale.DpiScaleY); 
         return m; 
     } 
 } 
```

性能上以上的计算可能比从字典获取的性能更好，不过这部分我没有测试

## 修复方法

最佳修复方法，等待 WPF 的大佬们合入我的修复，分发新的 dotnet 版本，更新版本即可

我所在的团队也分发了私有的 WPF 版本，包含此修复，如果大家也遇到此问题，且等不及我的修复合入主干，可以试试我所在的团队分发的版本，请看 [https://www.nuget.org/packages/dotnetCampus.WPF/6.0.4-alpha06-test02](https://www.nuget.org/packages/dotnetCampus.WPF/6.0.4-alpha06-test02)

## 更多文档

更多 DPI 相关请参阅

- [支持 Windows 10 最新 PerMonitorV2 特性的 WPF 多屏高 DPI 应用开发 - walterlv](https://blog.walterlv.com/post/windows-high-dpi-development-for-wpf.html )
- [Windows 下的高 DPI 应用开发（UWP / WPF / Windows Forms / Win32） - walterlv](https://blog.walterlv.com/post/windows-high-dpi-development.html )
- [Windows DPI Awareness for WPF - walterlv](https://blog.walterlv.com/windows/2014/09/20/windows-dpi-awareness-for-wpf.html )

更多触摸请参阅 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
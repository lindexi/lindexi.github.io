
# WPF 已知问题 在 WIC 层处理异常图片时 可能由于出现未处理异常导致进程退出

本文记录一个已知问题，此问题预计和 WPF 只有一毛钱关系，本质问题是在 WIC 层的 WindowsCodecs.dll 或 CLR 层上。在一些奇怪的系统上，解码一些奇怪的图片时，可能在解码器层抛出未捕获的本机异常，从而导致进程退出

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

我使用 ProcDump 工具抓到了一台服务器上 WPF 应用程序打开某个图片文件时，进程崩溃的问题，通过将 DUMP 拖入到 VisualStudio 可以看到异常提示信息如下

0x70B087F8 (WindowsCodecs.dll) (Foo.exe_231204_162615.dmp) Handled Exception: 0xC0000005: Read 0xFFFFFFFF ACCESS_VIOLATION.

以上的代码里面的 `0xC0000005` 表示 CLR 未知异常，在本文的情况下需要看更具体的异常。通过如下调用堆栈等信息，可以看到是在 `WindowsCodecs.dll!CScalerFant::ScaleXByteOneChannelLargeDownsample_SSE2` 里抛出本机异常 `Read 0xFFFFFFFF ACCESS_VIOLATION.` 错误。看起来就是在 WindowsCodecs.dll 里有一个实现上的 bug 导致越界之类

```
>	WindowsCodecs.dll!CScalerFant::ScaleXByteOneChannelLargeDownsample_SSE2(void *,unsigned int)
 	WindowsCodecs.dll!CScalerFant::ScaleYCommon_SSE()
 	WindowsCodecs.dll!CScalerFant::ScaleYByteChannel_SSE2()
 	WindowsCodecs.dll!CScalerFant::CopyPixels()
 	WindowsCodecs.dll!CBitmapScaler::CopyPixels()
 	WindowsCodecs.dll!CClipper::CopyPixels()
 	WindowsCodecs.dll!CSystemMemoryBitmap::HrInit()
 	WindowsCodecs.dll!HrCreateBitmapFromSource()
 	WindowsCodecs.dll!MILCreateBitmapFromSource()
 	WindowsCodecs.dll!WICCreateBitmapFromSource()
 	WindowsCodecs.dll!CCodecFactory::CreateBitmapFromSource()
 	WindowsCodecs.dll!_IWICImagingFactory_CreateBitmapFromSource_Proxy@16()
 	[Manage to Native]	
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.DUCECompatiblePtr.get() Line 538	C#
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.UpdateBitmapSourceResource(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}, bool skipOnChannelCheck = true) Line 958	C#
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.UpdateResource(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}, bool skipOnChannelCheck = true) Line 908	C#
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.AddRefOnChannelCore(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}) Line 916	C#
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.System.Windows.Media.Composition.DUCE.IResource.AddRefOnChannel(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}) Line 923	C#
 	PresentationCore.dll!System.Windows.Media.RenderData.System.Windows.Media.Composition.DUCE.IResource.AddRefOnChannel(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}) Line 178	C#
 	PresentationCore.dll!System.Windows.UIElement.RenderContent(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, bool isOnChannel = false) Line 6369	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateContent(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.VisualProxyFlags flags = IsSubtreeDirtyForRender | IsTransformDirty | IsClipDirty | IsContentDirty | IsOpacityDirty | IsOpacityMaskDirty | IsOffsetDirty | IsClearTypeHintDirty | IsGuidelineCollectionDirty | IsEdgeModeDirty | IsBitmapScalingModeDirty | IsEffectDirty | IsCacheModeDirty | IsScrollableAreaClipDirty | IsTextRenderingModeDirty | IsTextHintingModeDirty, bool isOnChannel = false) Line 1464	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1251	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateChildren(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.Composition.DUCE.ResourceHandle handle) Line 1489	C#
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}) Line 1255	C#
 	PresentationCore.dll!System.Windows.Media.Visual.Render(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, uint childIndex = 0) Line 1214	C#
 	PresentationCore.dll!System.Windows.Media.VisualBrush.System.Windows.Media.ICyclicBrush.RenderForCyclicBrush(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}, bool skipChannelCheck = false) Line 110	C#
 	PresentationCore.dll!System.Windows.Media.MediaContext.RaiseResourcesUpdated()	C#
 	PresentationCore.dll!System.Windows.Media.MediaContext.Render(System.Windows.Media.ICompositionTarget resizedCompositionTarget)	C#
 	PresentationCore.dll!System.Windows.Media.MediaContext.RenderMessageHandlerCore(object resizedCompositionTarget)	C#
 	PresentationCore.dll!System.Windows.Media.MediaContext.RenderMessageHandler(object resizedCompositionTarget = null)	C#
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = null, int numArgs = 1)
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source = {System.Windows.Threading.Dispatcher}, System.Delegate callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = null, int numArgs = 1, System.Delegate catchHandler = null)
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.WrappedInvoke(System.Delegate callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = null, int numArgs = 1, System.Delegate catchHandler = null)
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeImpl()
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(object state = {System.Windows.Threading.DispatcherOperation})
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.CallbackWrapper(object obj = {MS.Internal.CulturePreservingExecutionContext.CultureAndContextManager})
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservingExecutionContext executionContext = {MS.Internal.CulturePreservingExecutionContext}, System.Threading.ContextCallback callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object state = {System.Windows.Threading.DispatcherOperation})
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.Invoke()
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.ProcessQueue()
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.WndProcHook(System.IntPtr hwnd = 0x006e075a, int msg = 49506, System.IntPtr wParam = 0x00000000, System.IntPtr lParam = 0x00000000, ref bool handled = false)
 	WindowsBase.dll!MS.Win32.HwndWrapper.WndProc(System.IntPtr hwnd = 0x006e075a, int msg, System.IntPtr wParam = 0x00000000, System.IntPtr lParam = 0x00000000, ref bool handled = false)
 	WindowsBase.dll!MS.Win32.HwndSubclass.DispatcherCallbackOperation(object o = {MS.Win32.HwndSubclass.DispatcherOperationCallbackParameter})
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = {MS.Win32.HwndSubclass.DispatcherOperationCallbackParameter}, int numArgs = 1)
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source = {System.Windows.Threading.Dispatcher}, System.Delegate callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = {MS.Win32.HwndSubclass.DispatcherOperationCallbackParameter}, int numArgs = 1, System.Delegate catchHandler = null)
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.WrappedInvoke(System.Delegate callback = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = {MS.Win32.HwndSubclass.DispatcherOperationCallbackParameter}, int numArgs = 1, System.Delegate catchHandler = null)
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading.DispatcherPriority priority = Send, System.TimeSpan timeout = {System.TimeSpan}, System.Delegate method = {Method = {System.Reflection.RuntimeMethodInfo}}, object args = {MS.Win32.HwndSubclass.DispatcherOperationCallbackParameter}, int numArgs = 1)
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.Invoke(System.Windows.Threading.DispatcherPriority priority = Send, System.Delegate method = {Method = {System.Reflection.RuntimeMethodInfo}}, object arg = {MS.Win32.HwndSubclass.DispatcherOperationCallbackParameter})
 	WindowsBase.dll!MS.Win32.HwndSubclass.SubclassWndProc(System.IntPtr hwnd = 0x006e075a, int msg = 49506, System.IntPtr wParam = 0x00000000, System.IntPtr lParam = 0x00000000)
 	[Native to Manage]	
 	user32.dll!__InternalCallWinProc@20()
 	user32.dll!UserCallWinProcCheckWow()
 	user32.dll!DispatchMessageWorker()
 	user32.dll!_DispatchMessageW@4()
 	[Manage to Native]	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.DispatcherFrame frame = {System.Windows.Threading.DispatcherFrame})
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.DispatcherFrame frame = {System.Windows.Threading.DispatcherFrame})
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.Run()
 	PresentationFramework.dll!System.Windows.Application.RunDispatcher(object ignore = null)
 	PresentationFramework.dll!System.Windows.Application.RunInternal(System.Windows.Window window = null)
 	PresentationFramework.dll!System.Windows.Application.Run(System.Windows.Window window = null)
 	PresentationFramework.dll!System.Windows.Application.Run()
 	Foo.Program.Main(string[] args = {string[5]})
```

这个 WindowsCodecs.dll 是属于系统的 WIC 组件，跟随系统版本和系统更新，我测试了图片在我的机器上，是可以正常使用的。换句话说就是这次的崩溃完全是被系统层组件带的

我所抓的系统是 Windows Server 2016 1607 14393.3808 版本，当我更新系统完成之后，也没有再复现此问题

会导致进程退出的原因是接收到了一个本机异常，在 dotnet core 的设计下，废除了 HandleProcessCorruptedStateExceptions 等机制，当收到本机异常时将会导致进程退出。详细请看 [升级到 dotnet core 之后 HandleProcessCorruptedStateExceptions 无法接住异常](https://blog.lindexi.com/post/%E5%8D%87%E7%BA%A7%E5%88%B0-dotnet-core-%E4%B9%8B%E5%90%8E-HandleProcessCorruptedStateExceptions-%E6%97%A0%E6%B3%95%E6%8E%A5%E4%BD%8F%E5%BC%82%E5%B8%B8.html )

我将此问题报告给 WPF 官方：https://github.com/dotnet/wpf/issues/8499

但是预估这个问题即使要解决也不是在 WPF 这一层解决。想想，要是你调用了某个系统组件，这个组件炸了，那你的应用要不要跟着炸，如果不跟着炸，会不会造成更大的危害，比如损坏数据等等

为什么 WIC 层系统组件存在问题会影响 WPF 应用程序？这是因为 WPF 的多媒体编码解码是通过 WIC 层实现的，详细请看 [dotnet 读 WPF 源代码笔记 WIC 多媒体图片处理通过 WindowsCodecs.dll 实现功能](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-WIC-%E5%A4%9A%E5%AA%92%E4%BD%93%E5%9B%BE%E7%89%87%E5%A4%84%E7%90%86%E9%80%9A%E8%BF%87-WindowsCodecs.dll-%E5%AE%9E%E7%8E%B0%E5%8A%9F%E8%83%BD.html )

为什么说此问题和 WPF 只有一毛钱关系？这是因为直接走 WIC 解码本身就有问题，不通过 WPF 自己手动调用 WIC 的方法也能复现，请看 [dotnet win32 使用 WIC 获取系统编解码器](https://blog.lindexi.com/post/dotnet-win32-%E4%BD%BF%E7%94%A8-WIC-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E7%BC%96%E8%A7%A3%E7%A0%81%E5%99%A8.html ) 或者是通过 DirectX 方式走，请看 [在 Direct2D 绘制从 WIC 加载的图片](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E5%9C%A8-Direct2D-%E7%BB%98%E5%88%B6%E4%BB%8E-WIC-%E5%8A%A0%E8%BD%BD%E7%9A%84%E5%9B%BE%E7%89%87.html )

更进一步，更新系统之后就不复现问题，也就是说很快某软就发现了这个问题，默默修了。由于我一口气更新了大量补丁，我不知道具体哪个补丁修复了这个问题

补充：可能你可以看到的中文提示大概如下

```
0x70B087F8 (WindowsCodecs.dll) (Foo.exe_231204_162615.dmp 中)处有未经处理的异常: 0xC0000005: 读取位置 0xFFFFFFFF 时发生访问冲突。
```

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

更多 WPF 已知问题请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
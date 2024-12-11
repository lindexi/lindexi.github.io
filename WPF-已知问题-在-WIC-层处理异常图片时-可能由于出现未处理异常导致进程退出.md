
# WPF 已知问题 在 WIC 层处理异常图片时 可能由于出现未处理异常导致进程退出

本文记录一个已知问题，此问题预计和 WPF 只有一毛钱关系，本质问题是在 WIC 层的 WindowsCodecs.dll 或 CLR 层上。在一些奇怪的系统上，解码一些奇怪的图片时，可能在解码器层抛出未捕获的本机异常，从而导致进程退出

<!--more-->


<!-- CreateTime:2023/12/5 11:04:31 -->

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

---

今天 2024.12.11 又有用户反馈了类似的问题

具体表现就是打开软件，软件碰到某张图片就崩溃，进程直接崩溃，没有任何日志。一般遇到这个问题，首先猜测的就是 WIC 层相关问题

这张图片是一张灰度图，下载地址： [点此下载](https://pinco.seewo.com/s/4554f5af7af64e80b4c56a8804e3375d)

使用 MediaInfo 工具查看，可见信息如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<MediaInfo xmlns="https://mediaarea.net/mediainfo" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://mediaarea.net/mediainfo https://mediaarea.net/mediainfo/mediainfo_2_0.xsd" version="2.0">
    <creatingLibrary version="22.12" url="https://mediaarea.net/MediaInfo">MediaInfoLib</creatingLibrary>
    <media ref="E:\lindexi\测试文件\测试图片\灰度图090e2b99c4eb8349580b2c07a3ca9941.png">
        <track type="General">
            <ImageCount>1</ImageCount>
            <FileExtension>png</FileExtension>
            <Format>PNG</Format>
            <FileSize>13518</FileSize>
            <StreamSize>0</StreamSize>
            <File_Created_Date>UTC 2024-12-11 03:33:41.993</File_Created_Date>
            <File_Created_Date_Local>2024-12-11 11:33:41.993</File_Created_Date_Local>
            <File_Modified_Date>UTC 2024-12-11 02:41:57.106</File_Modified_Date>
            <File_Modified_Date_Local>2024-12-11 10:41:57.106</File_Modified_Date_Local>
        </track>
        <track type="Image">
            <Format>PNG</Format>
            <Format_Compression>Deflate</Format_Compression>
            <Width>1088</Width>
            <Height>624</Height>
            <ColorSpace>Y</ColorSpace>
            <BitDepth>1</BitDepth>
            <Compression_Mode>Lossless</Compression_Mode>
            <StreamSize>13518</StreamSize>
        </track>
    </media>
</MediaInfo>
```

以上的核心参数是：

```xml
<ColorSpace>Y</ColorSpace>
<BitDepth>1</BitDepth>
<Compression_Mode>Lossless</Compression_Mode>
```

用户设备上的 WindowsCodecs.dll 的版本号是 10.0.14393.7513

用了 dism 尝试修复，命令如下

```
dism /online /Cleanup-Image /RestoreHealth
```

再尝试系统更新 KB5048671 和 KB5046612 补丁

补充：

抓取此问题的 dump 的方法如下

由于此问题会最终落到 CLR 崩溃处理上，经过了 CLR 崩溃处理之后再退出进程，不能直接使用以下命令抓取

```
procdump -e -t -ma 进程Id号
```

或者配置注册表 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting` 设置进程退出就自动生成 DUMP 文件，以上这两个方式所获取的都是废料文件，因为已经错过了第一次的崩溃异常了，只能拿到一个空壳的文件

正确抓取的方法是设置 `-e 1` 第一次机会异常，再配合输出的 C0000005 异常，使用以下命令抓取

```
procdump -t -ma -e 1 -f C0000005 进程Id号
```

以上命令的参数的含义如下：

-t	进程终止时写入转储。

-ma	写入“完整”转储文件。

-e	当进程遇到未经处理的异常时写入转储。包含 1 以在第一次出现异常时创建转储。合起来 `-e 1` 就是第一次机会异常。我开始忘记加上了这个，导致错过了异常，抓取到了事后现场，无法分析

-f 筛选（包括）DLL 加载/卸载时的异常内容、调试日志记录和文件名。 支持通配符 (`*`)

抓取步骤是先使用 `procdump -t -ma -e 进程Id号` 命令抓，发现抓到的是废料。再看原来的输出，有哪些 `Exception: ` 输出信息，从后向前，将其信息代入到 `-f` 参数里面，使用 `procdump -t -ma -e 1 -f 带入的信息 进程Id号` 格式的命令抓取

比如一开始使用 `procdump -t -ma -e 进程Id号` 命令抓的时候，输出的信息如下

```
[10:23:30] Exception: 000006BA
[10:23:31] Exception: C0000005.ACCESS_VIOLATION
[10:23:32] Process Exit: PID 123, Exit Code 0xc0000005
```

按照从后向前的顺序，先试试 `[10:23:31] Exception: C0000005.ACCESS_VIOLATION` 这条信息，代入 `-f` 参数的时候，只取关键少量部分，这里的 `-f` 参数最终会做字符串匹配，宁可少字也不要错字哈。代入之后的命令如下

```
procdump -t -ma -e 1 -f C0000005 进程Id号
```

假设此时的异常信息抓取到的依然没有帮助，那证明这个异常还不是咱想要的，配合时间看，似乎前面的 `[10:23:30] Exception: 000006BA` 才是关键，继续抓取。代入之后的命令如下

```
procdump -t -ma -e 1 -f 000006BA 进程Id号
```

大概简单的抓取方法就是这样了

为什么要从后向前逐个代入呢？因为有些异常是可以被捕获的，比如上述例子的 `000006BA` 异常，可能只是一个内部的 RPC 异常，异常信息如下

```
0x000006BA: RPC 服务器不可用
```

错误堆栈如下

```
>	KERNELBASE.dll!_RaiseException@16()	未知
 	rpcrt4.dll!RpcpRaiseException()	未知
 	rpcrt4.dll!_RpcRaiseException@4()	未知
 	rpcrt4.dll!_NdrClientCall2()	未知
 	winsta.dll!_RpcGetCurrentSessionProtocolLastInputTime@20()	未知
 	winsta.dll!GetCurrentSessionInformation(struct _WINSTATIONINFORMATIONW *,unsigned long,unsigned long *)	未知
 	winsta.dll!_WinStationQueryCurrentSessionInformation@16()	未知
 	winsta.dll!_WinStationQueryInformationW@24()	未知
 	wtsapi32.dll!WTSQuerySessionInformationW()	未知
 	[托管到本机的转换]	
 	WindowsBase.dll!MS.Win32.SafeNativeMethods.IsCurrentSessionConnectStateWTSActive(int? SessionId = 1, bool defaultResult = true)	未知
 	PresentationCore.dll!System.Windows.Interop.HwndTarget.HwndTarget(System.IntPtr hwnd = 0x000d085c)	未知
 	PresentationCore.dll!System.Windows.Interop.HwndSource.Initialize(System.Windows.Interop.HwndSourceParameters parameters)	未知
 	PresentationCore.dll!System.Windows.Interop.HwndSource.HwndSource(System.Windows.Interop.HwndSourceParameters parameters = {System.Windows.Interop.HwndSourceParameters})	未知
 	PresentationFramework.dll!System.Windows.Window.CreateSourceWindow(bool duringShow = true)	未知
 	PresentationFramework.dll!System.Windows.Window.CreateSourceWindowDuringShow()	未知
 	PresentationFramework.dll!System.Windows.Window.SafeCreateWindowDuringShow()	未知
 	PresentationFramework.dll!System.Windows.Window.ShowHelper(object booleanBox = true)	未知
 	PresentationFramework.dll!System.Windows.Window.Show()	未知
>	KERNELBASE.dll!_RaiseException@16()	未知
 	rpcrt4.dll!RpcpRaiseException()	未知
 	rpcrt4.dll!_RpcRaiseException@4()	未知
 	rpcrt4.dll!_NdrClientCall2()	未知
 	winsta.dll!_RpcGetCurrentSessionProtocolLastInputTime@20()	未知
 	winsta.dll!GetCurrentSessionInformation(struct _WINSTATIONINFORMATIONW *,unsigned long,unsigned long *)	未知
 	winsta.dll!_WinStationQueryCurrentSessionInformation@16()	未知
 	winsta.dll!_WinStationQueryInformationW@24()	未知
 	wtsapi32.dll!WTSQuerySessionInformationW()	未知
 	[托管到本机的转换]	
 	WindowsBase.dll!MS.Win32.SafeNativeMethods.IsCurrentSessionConnectStateWTSActive(int? SessionId = 1, bool defaultResult = true)	未知
 	PresentationCore.dll!System.Windows.Interop.HwndTarget.HwndTarget(System.IntPtr hwnd = 0x000d085c)	未知
 	PresentationCore.dll!System.Windows.Interop.HwndSource.Initialize(System.Windows.Interop.HwndSourceParameters parameters)	未知
 	PresentationCore.dll!System.Windows.Interop.HwndSource.HwndSource(System.Windows.Interop.HwndSourceParameters parameters = {System.Windows.Interop.HwndSourceParameters})	未知
 	PresentationFramework.dll!System.Windows.Window.CreateSourceWindow(bool duringShow = true)	未知
 	PresentationFramework.dll!System.Windows.Window.CreateSourceWindowDuringShow()	未知
 	PresentationFramework.dll!System.Windows.Window.SafeCreateWindowDuringShow()	未知
 	PresentationFramework.dll!System.Windows.Window.ShowHelper(object booleanBox = true)	未知
 	PresentationFramework.dll!System.Windows.Window.Show()	未知
```

这个异常是发生在 WPF 调用 IsCurrentSessionConnectStateWTSActive 方法，即 [`wtsapi32.dll!WTSQuerySessionInformationW`](https://learn.microsoft.com/zh-cn/windows/win32/api/wtsapi32/nf-wtsapi32-wtsquerysessioninformationw) 方法时，在 Win32 内部出现的异常。这个异常很快被抓住处理，不会导致最终崩溃。最终只会在 WTSQuerySessionInformationW 方法调用返回值，返回失败而已

从理论分析，这个方法的调用发生在每次窗口的 Show 里面，意味着影响范围是全部窗口。如果窗口已经有正常显示过的，那基本上不会是这里再次导致崩溃。直接抓取这个 dump 文件分析，就会被带偏思路

这就是为什么说为了减少干扰，就推荐大家从后到前的顺序代入抓取的原因。越后面的异常才越可能是导致崩溃的异常，前面的异常可能是被正确抓取处理的

更多请参阅 <https://learn.microsoft.com/zh-cn/sysinternals/downloads/procdump>




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
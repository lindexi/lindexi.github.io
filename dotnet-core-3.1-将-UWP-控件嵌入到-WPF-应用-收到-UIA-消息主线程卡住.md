
# dotnet core 3.1 将 UWP 控件嵌入到 WPF 应用 收到 UIA 消息主线程卡住

本文记录一个问题，此问题是在 .NET Core 3.1 的 WPF 应用里面，嵌入 UWP 控件之后，在收到 UIA 的消息时，可能让主线程卡住。暂时此问题还不知道具体的复现步骤，此问题不确定是否真的和 WPF 或 UWP 相关，此问题可能仅仅只是 UIA 模块的问题

<!--more-->


<!-- 博客 -->
<!-- 发布 -->

此问题没有在 dotnet 6 应用进行测试，这是因为 dotnet 6 和 .NET Core 3.1 在 UWP 嵌入方面存在很大的不同，同时此问题也没有找到复现步骤

这个问题是我的小伙伴告诉我的，十分有趣的是这个问题只有在她的某个客户的机器上才能复现，而且那个客户还能气人不给远程。我所拿到的信息只有一些 DUMP 文件，本文以下都是通过 DUMP 进行分析所猜测的。能够知道导致 WPF 的主线程卡住的原因，但是不知道这个问题的根本原因

我将此问题报告给 WPF 官方，请看 https://github.com/dotnet/wpf/issues/8347

复现问题的条件如下

- 创建一个 .NET Core 3.1 的 WPF 项目
- 随意嵌入 UWP 控件
- 在一个有 UIA 存在的环境下运行项目


有 UIA 存在的环境指的是有使用 UIA 模块的进程存在的系统环境，比如运行了 PAD 或 AxeWindows 程序。但我不知道这个问题的具体复现步骤，只是这么做的话，可以就可以让主线程卡顿。但我自己重复了许多次都没有复现

主线程卡顿的堆栈如下

```
00 0285bbd8 77365e03 win32u!NtUserPostMessage+0xc
01 0285bbf8 760633f1 user32!PostMessageW+0x53
02 0285bc20 761be28a combase!CComApartment::ClassicSTAPostMessage(unsigned int msg = 0x403, class IMessageParam * pParam = 0x0e5add18)+0x32 [onecore\com\combase\dcomrem\classicsta.cpp @ 150] 
03 0285bc34 761ae970 combase!CComApartment::STAPostReleaseRifRef(class IMessageParam * pParam = 0x0e5add18)+0x19 [onecore\com\combase\dcomrem\aprtmnt.cxx @ 1451] 
04 0285bc60 7617c671 combase!PostReleaseRifRef(struct IRemUnknown * pRemUnk = 0x0e48033c, int fReleaseRemUnkProxy = 0n0, class OXIDEntry * pOXIDEntry = 0x076d1238, unsigned short cRifRef = 1, struct tagREMINTERFACEREF * pRifRef = 0x0e5ad51c, struct IUnknown * pAsyncRelease = 0x00000000)+0xee [onecore\com\combase\dcomrem\marshal.cxx @ 8866] 
05 0285bc9c 761ae7a6 combase!RemoteReleaseRifRefHelper+0x8b916 [onecore\com\combase\dcomrem\marshal.cxx @ 8740] 
06 0285bcd0 76176afd combase!HandlePostReleaseRifRef(class IMessageParam * pMessageParam = 0x00000000)+0x96 [onecore\com\combase\dcomrem\marshal.cxx @ 8938] 
07 0285bd64 7737150b combase!ThreadWndProc+0x9160d [onecore\com\combase\dcomrem\chancont.cxx @ 764] 
08 0285bd90 773684ba user32!_InternalCallWinProc+0x2b
09 0285be74 7736622a user32!UserCallWinProcCheckWow+0x33a
0a 0285bee8 77365fe0 user32!DispatchMessageWorker+0x22a
0b 0285bef4 760f2330 user32!DispatchMessageW+0x10
0c (Inline) -------- combase!CCliModalLoop::MyDispatchMessage(void)+0xa [onecore\com\combase\dcomrem\callctrl.cxx @ 2989] 
0d 0285bf28 760f1bbf combase!CCliModalLoop::PeekRPCAndDDEMessage(void)+0x4d [onecore\com\combase\dcomrem\callctrl.cxx @ 2616] 
0e 0285bf94 760f17b4 combase!CCliModalLoop::BlockFn(void ** ahEvent = 0x04bdd728, unsigned long cEvents = 1, unsigned long * lpdwSignaled = 0x0285c0a0)+0x1fd [onecore\com\combase\dcomrem\callctrl.cxx @ 2103] 
0f 0285c054 760f11e7 combase!ClassicSTAThreadWaitForHandles(unsigned long dwFlags = 2, unsigned long dwTimeout = 1, unsigned long cHandles = 1, void ** pHandles = 0x04bdd728, unsigned long * pdwIndex = 0x0285c0a0)+0xb4 [onecore\com\combase\dcomrem\classicsta.cpp @ 51] 
10 0285c080 7a58bb8c combase!CoWaitForMultipleHandles(unsigned long dwFlags = 2, unsigned long dwTimeout = 1, unsigned long cHandles = 1, void ** pHandles = 0x04bdd728, unsigned long * lpdwindex = 0x0285c0a0)+0x77 [onecore\com\combase\dcomrem\sync.cxx @ 122] 
11 0285c0a4 7a54e7bf coreclr!MsgWaitHelper(int numWaiters = 0n42320032, void ** phEvent = <Value unavailable error>, int bWaitAll = 0n0, unsigned long millis = 1, int bAlertable = 0n1)+0x40 [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3318] 
12 0285c0bc 7a44b0cc coreclr!Thread::DoAppropriateAptStateWait+0x103631 [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3364] 
13 0285c154 7a44aec7 coreclr!Thread::DoAppropriateWaitWorker(int countHandles = 0n1, void ** handles = 0x04bdd728, int waitAll = 0n0, unsigned long millis = 1, WaitMode mode = 0n3 (No matching enumerant))+0x1ab [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3507] 
14 0285c1d4 7a4494e3 coreclr!Thread::DoAppropriateWait(int countHandles = 0n1, void ** handles = 0x04bdd728, int waitAll = 0n0, unsigned long millis = 1, WaitMode mode = 0n3 (No matching enumerant), struct PendingSync * syncState = 0x00000000)+0x79 [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3203] 
15 0285c274 7abf6c25 coreclr!WaitHandleNative::CorWaitMultipleNative(void ** handleArray = 0x04bdd728, int numHandles = 0n1, int timeout = 0n1, bool waitForAll = false)+0x63 [d:\workspace\_work\1\s\src\vm\comwaithandle.cpp @ 57] 
16 0285c290 7bf8b55e System_Private_CoreLib!System.Threading.SynchronizationContext.WaitHelper+0x31
17 0285c2a0 7abf6a5f WindowsBase!System.Windows.Threading.DispatcherSynchronizationContext.Wait+0x46
18 0285c2b0 7a4aa0ef System_Private_CoreLib!System.Threading.SynchronizationContext.InvokeWaitMethodHelper+0x17
19 0285c2c4 7a42ff91 coreclr!CallDescrWorkerInternal(unsigned long pParams = 0x285c32c)+0x34 [D:\workspace\_work\1\s\src\vm\i386\asmhelpers.asm @ 615] 
1a (Inline) -------- coreclr!CallDescrWorkerWithHandler(void)+0x55 [d:\workspace\_work\1\s\src\vm\callhelpers.cpp @ 70] 
1b 0285c358 7a58b67e coreclr!MethodDescCallSite::CallTargetWorker(unsigned int64 * pArguments = 0x0285c3fc, unsigned int64 * pReturnValue = 0x0285c3ec, int cbReturnValue = 0n8)+0x167 [d:\workspace\_work\1\s\src\vm\callhelpers.cpp @ 604] 
1c (Inline) -------- coreclr!MethodDescCallSite::Call_RetI4(void)+0x15 [d:\workspace\_work\1\s\src\vm\callhelpers.h @ 468] 
1d 0285c43c 7a54e5ab coreclr!Thread::DoSyncContextWait(class Object ** pSyncCtxObj = 0x0285c4c0, int countHandles = 0n1, void ** handles = 0x0285c57c, int waitAll = 0n0, unsigned long millis = 1)+0xad [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3926] 
1e 0285c4d4 7a44aec7 coreclr!Thread::DoAppropriateWaitWorker+0x10368a [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3471] 
1f 0285c554 7a58bb26 coreclr!Thread::DoAppropriateWait(int countHandles = 0n1, void ** handles = 0x0285c57c, int waitAll = 0n0, unsigned long millis = 1, WaitMode mode = WaitMode_Alertable (0n1), struct PendingSync * syncState = 0x00000000)+0x79 [d:\workspace\_work\1\s\src\vm\threads.cpp @ 3203] 
20 0285c580 7a54874a coreclr!Thread::JoinEx(unsigned long timeout = 1, WaitMode mode = WaitMode_Alertable (0n1))+0x4f [d:\workspace\_work\1\s\src\vm\threads.cpp @ 388] 
21 (Inline) -------- coreclr!Thread::Join+0x9 [d:\workspace\_work\1\s\src\vm\threads.cpp @ 363] 
22 0285c5cc 7a44045f coreclr!RCWCleanupList::CleanupWrappersInCurrentCtxThread+0x1082e3 [d:\workspace\_work\1\s\src\vm\runtimecallablewrapper.cpp @ 1799] 
23 0285ca40 7a43fe64 coreclr!RCW::Initialize(struct IUnknown * pUnk = 0x00000000, unsigned long dwSyncBlockIndex = 0x285c608, class MethodTable * pClassMT = 0x06ced214)+0x1b9 [d:\workspace\_work\1\s\src\vm\runtimecallablewrapper.cpp @ 2355] 
24 0285ca94 7a43fd58 coreclr!RCW::CreateRCWInternal(struct IUnknown * pUnk = 0x0e565c00, unsigned long dwSyncBlockIndex = 0x152, unsigned long flags = 1, class MethodTable * pClassMT = 0x06ced214)+0xb8 [d:\workspace\_work\1\s\src\vm\runtimecallablewrapper.cpp @ 2158] 
25 0285cacc 7a43f9bf coreclr!RCW::CreateRCW(struct IUnknown * pUnk = 0x0e565c00, unsigned long dwSyncBlockIndex = 0x152, unsigned long flags = 1, class MethodTable * pClassMT = 0x06ced214)+0x58 [d:\workspace\_work\1\s\src\vm\runtimecallablewrapper.cpp @ 2090] 
26 0285cb68 7a43f904 coreclr!COMInterfaceMarshaler::CreateObjectRef(int fDuplicate = 0n0, class Object ** pComObj = 0x0285cbec, struct IUnknown ** ppIncomingIP = 0x0285cc68, class MethodTable * pIncomingItfMT = 0x0dc98944, bool bIncomingIPAddRefed = false)+0xa7 [d:\workspace\_work\1\s\src\vm\cominterfacemarshaler.cpp @ 289] 
27 0285cbfc 7a498941 coreclr!COMInterfaceMarshaler::FindOrCreateObjectRefInternal(struct IUnknown ** ppIncomingIP = 0x0285cc68, class MethodTable * pIncomingItfMT = 0x0dc98944, bool bIncomingIPAddRefed = false)+0x14b [d:\workspace\_work\1\s\src\vm\cominterfacemarshaler.cpp @ 901] 
28 (Inline) -------- coreclr!COMInterfaceMarshaler::FindOrCreateObjectRef(struct IUnknown * pIncomingIP = 0x0e565c00)+0x1a [d:\workspace\_work\1\s\src\vm\cominterfacemarshaler.cpp @ 819] 
29 0285d0a4 7a4987e4 coreclr!GetObjectRefFromComIP(class Object ** pObjOut = 0x0285d144, struct IUnknown ** ppUnk = <Value unavailable error>, class MethodTable * pMTClass = 0x00000000, class MethodTable * pItfMT = 0x0dc98944, unsigned long dwFlags = 0)+0x144 [d:\workspace\_work\1\s\src\vm\interopconverter.cpp @ 529] 
2a 0285d0c4 7a498742 coreclr!UnmarshalObjectFromInterface(class Object ** ppObjectDest = 0x0285d144, struct IUnknown ** ppUnkSrc = <Value unavailable error>, class MethodTable * pItfMT = 0x0dc98944, class MethodTable * pClassMT = 0x00000000, unsigned long dwFlags = 0)+0x37 [d:\workspace\_work\1\s\src\vm\interoputil.cpp @ 6345] 
2b 0285d158 0cf7af6c coreclr!StubHelpers::InterfaceMarshaler__ConvertToManaged(struct IUnknown ** ppUnk = 0x0285d190, class MethodTable * pItfMT = 0x0dc98944, unsigned long dwFlags = 0, class MethodTable * pClsMT = 0x00000000)+0xd2 [d:\workspace\_work\1\s\src\vm\stubhelpers.cpp @ 16707566] 
WARNING: Frame IP not in any known module. Following frames may be wrong.
2c 0285d1b8 0dc39b12 0xcf7af6c
2d 0285d1c8 08213ca2 UIAutomationProvider!System.Windows.Automation.Provider.AutomationInteropProvider.HostProviderFromHandle+0x32
2e 0285d1e0 0cf7ae4c PresentationCore!MS.Internal.Automation.ElementProxy.get_HostRawElementProvider+0x56
2f 0285d210 7a4aa36a 0xcf7ae4c
30 0285d268 046fd586 coreclr!COMToCLRDispatchHelper(void)+0x28 [D:\workspace\_work\1\s\src\vm\i386\asmhelpers.asm @ 1330] 
31 0285d294 0edf3f85 0x46fd586
32 0285d328 0edf255d UIAutomationCore!UiaUtils::EntryPointFromImmediateProvider+0xb5
33 0285d5d8 0edf1b06 UIAutomationCore!UiaNodeFactory::CompleteNodeInfo+0xad
34 0285d5f4 0ee1c01f UIAutomationCore!UiaNodeFactory::FromPartialNodeInfo+0x16
35 0285d694 0ee1c0e3 UIAutomationCore!UiaNodeFactory::FromLocalProvider+0x18a
36 0285d6b4 0edc070d UIAutomationCore!UiaNodeFactory::FromLocalProvider+0x1b
37 0285e73c 0cf77713 UIAutomationCore!UiaReturnRawElementProvider+0x4ed
38 0285e7a4 0dc39b5e 0xcf77713
39 0285e7c0 082f04f2 UIAutomationProvider!System.Windows.Automation.Provider.AutomationInteropProvider.ReturnRawElementProvider+0x36
3a 0285e7f4 082efd67 PresentationCore!System.Windows.Interop.HwndTarget.CriticalHandleWMGetobject+0x56
3b 0285e8a8 082eb1fb PresentationCore!System.Windows.Interop.HwndTarget.HandleMessage+0x5d7
3c 0285e8c0 7bf30a5c PresentationCore!System.Windows.Interop.HwndSource.HwndTargetFilterMessage+0x33
3d 0285e900 7bf30054 WindowsBase!MS.Win32.HwndWrapper.WndProc+0x80
3e 0285e920 7bf873d3 WindowsBase!MS.Win32.HwndSubclass.DispatcherCallbackOperation+0x5c
3f 0285e940 7bf872a3 WindowsBase!System.Windows.Threading.ExceptionWrapper.InternalRealCall+0xd3
40 0285e97c 7bf89233 WindowsBase!System.Windows.Threading.ExceptionWrapper.TryCatchWhen+0x37
41 0285e9dc 7bf2feb1 WindowsBase!System.Windows.Threading.Dispatcher.LegacyInvokeImpl+0x177
42 0285ea5c 046fd1e9 WindowsBase!MS.Win32.HwndSubclass.SubclassWndProc+0x1cd
43 0285ea90 7737150b 0x46fd1e9
44 0285eabc 773684ba user32!_InternalCallWinProc+0x2b
45 0285eba0 773680ba user32!UserCallWinProcCheckWow+0x33a
46 0285ec04 7736bc7f user32!DispatchClientMessage+0xea
47 0285ec40 77af503d user32!__fnDWORD+0x3f
48 0285ec78 7736bcde ntdll!KiUserCallbackDispatcher+0x4d
49 0285ecb4 76367bb9 user32!GetMessageW+0x2e
4a 0285ecd4 258d5032 msctf!CThreadInputMgr::GetMessageW+0x29
4b 0285ed50 7bf8a455 0x258d5032
4c 0285ed9c 7bf8a322 WindowsBase!System.Windows.Threading.Dispatcher.GetMessage+0x85
4d 0285edf0 7bf880b2 WindowsBase!System.Windows.Threading.Dispatcher.PushFrameImpl+0x7e
4e 0285edfc 7bf88078 WindowsBase!System.Windows.Threading.Dispatcher.PushFrame+0x36
4f 0285ee08 7b6f26ab WindowsBase!System.Windows.Threading.Dispatcher.Run+0x38
50 0285ee14 7b6f13d0 PresentationFramework!System.Windows.Application.RunDispatcher+0x1b
51 0285ee30 7b6eee85 PresentationFramework!System.Windows.Application.RunInternal+0x12c
52 0285ee3c 06f70cd7 PresentationFramework!System.Windows.Application.Run+0x29
53 0285ee68 7a4aa0ef 0x6f70cd7
54 0285ee74 7a42ff91 coreclr!CallDescrWorkerInternal(unsigned long pParams = 0x285eed8)+0x34 [D:\workspace\_work\1\s\src\vm\i386\asmhelpers.asm @ 615] 
55 (Inline) -------- coreclr!CallDescrWorkerWithHandler(void)+0x55 [d:\workspace\_work\1\s\src\vm\callhelpers.cpp @ 70] 
56 0285ef04 7a46c895 coreclr!MethodDescCallSite::CallTargetWorker(unsigned int64 * pArguments = 0x0285f048, unsigned int64 * pReturnValue = 0x00000000, int cbReturnValue = 0n0)+0x167 [d:\workspace\_work\1\s\src\vm\callhelpers.cpp @ 604] 
57 (Inline) -------- coreclr!MethodDescCallSite::Call(void)+0x11 [d:\workspace\_work\1\s\src\vm\callhelpers.h @ 468] 
58 0285f080 7a46e292 coreclr!RunMain(class MethodDesc * pFD = 0x06f5eed8, int * piRetVal = 0x0285f0b4, class PtrArray ** stringArgs = 0x0285f3b8)+0x1e4 [d:\workspace\_work\1\s\src\vm\assembly.cpp @ 1558] 
59 0285f2e4 7a46de94 coreclr!Assembly::ExecuteMainMethod(class PtrArray ** stringArgs = 0x0285f3b8)+0x178 [d:\workspace\_work\1\s\src\vm\assembly.cpp @ 1681] 
5a 0285f3d0 7a46dcb3 coreclr!CorHost2::ExecuteAssembly(unsigned long dwAppDomainId = 1, wchar_t * pwzAssemblyPath = 0x047d0808 "C:\Program Files (x86)\Lindexi\App\Lindexi_3.0.0.224\UWP\Lindexi.dll", int argc = 0n0, wchar_t ** argv = 0x00000000, unsigned long * pReturnValue = 0x0285f4a4)+0x184 [d:\workspace\_work\1\s\src\vm\corhost.cpp @ 460] 
5b 0285f424 602246e6 coreclr!coreclr_execute_assembly(void * hostHandle = 0x02bfc99c, unsigned int domainId = 1, int argc = 0n0, char ** argv = 0x00000000, char * managedAssemblyPath = 0x047ea9a0 "C:\Program Files (x86)\Lindexi\App\Lindexi_3.0.0.224\UWP\Lindexi.dll", unsigned int * exitCode = 0x0285f4a4)+0xd3 [d:\workspace\_work\1\s\src\dlls\mscoree\unixinterface.cpp @ 412] 
5c 0285f448 60232e1d hostpolicy!coreclr_t::execute_assembly(int argc = 0n0, char ** argv = 0x00000000, char * managed_assembly_path = 0x047ea9a0 "C:\Program Files (x86)\Lindexi\App\Lindexi_3.0.0.224\UWP\Lindexi.dll", unsigned int * exit_code = 0x0285f4a4)+0x26 [d:\workspace\_work\1\s\src\corehost\cli\hostpolicy\coreclr.cpp @ 145] 
5d 0285f4f8 60232a87 hostpolicy!run_app_for_context(struct hostpolicy_context_t * context = 0x02b67620, int argc = 0n0, wchar_t ** argv = 0x00000000)+0x33d [d:\workspace\_work\1\s\src\corehost\cli\hostpolicy\hostpolicy.cpp @ 247] 
5e 0285f530 60233bd1 hostpolicy!run_app(int argc = 0n0, wchar_t ** argv = 0x02b54b9c)+0x57 [d:\workspace\_work\1\s\src\corehost\cli\hostpolicy\hostpolicy.cpp @ 276] 
5f 0285f638 602bd119 hostpolicy!corehost_main(int argc = 0n1, wchar_t ** argv = 0x02b54b98)+0xf1 [d:\workspace\_work\1\s\src\corehost\cli\hostpolicy\hostpolicy.cpp @ 390] 
60 0285f6ac 602bfa46 hostfxr!execute_app(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * impl_dll_dir = 0x0285f70c "C:\Program Files (x86)\dotnet\shared\Microsoft.NETCore.App\3.1.19", class corehost_init_t * init = 0x02b69e38, int argc = 0n1, wchar_t ** argv = 0x02b54b98)+0x309 [d:\workspace\_work\1\s\src\corehost\cli\fxr\fx_muxer.cpp @ 147] 
61 0285f734 602beaee hostfxr!`anonymous namespace'::read_config_and_execute(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * host_command = 0x0285f8b4 "", struct host_startup_info_t * host_info = 0x0285f8f4, class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * app_candidate = 0x0285f884 "C:\Program Files (x86)\Lindexi\App\Lindexi_3.0.0.224\UWP\Lindexi.dll", class std::unordered_map<enum known_options,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > >,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > > > > > * opts = 0x0285f7dc { size=0x0 }, int new_argc = 0n1, wchar_t ** new_argv = 0x02b54b98, host_mode_t mode = apphost (0n2), wchar_t * out_buffer = 0x00000000 "", int buffer_size = 0n0, int * required_buffer_size = 0x00000000)+0xa6 [d:\workspace\_work\1\s\src\corehost\cli\fxr\fx_muxer.cpp @ 502] 
62 0285f798 602bcd55 hostfxr!fx_muxer_t::handle_exec_host_command(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * host_command = 0x0285f8b4 "", struct host_startup_info_t * host_info = 0x0285f8f4, class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * app_candidate = 0x0285f884 "C:\Program Files (x86)\Lindexi\App\Lindexi_3.0.0.224\UWP\Lindexi.dll", class std::unordered_map<enum known_options,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > >,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > > > > > * opts = 0x0285f7dc { size=0x0 }, int argc = 0n1, wchar_t ** argv = 0x02b54b98, int argoff = 0n1, host_mode_t mode = apphost (0n2), wchar_t * result_buffer = 0x00000000 "", int buffer_size = 0n0, int * required_buffer_size = 0x00000000)+0xee [d:\workspace\_work\1\s\src\corehost\cli\fxr\fx_muxer.cpp @ 952] 
63 0285f8ac 602b93c6 hostfxr!fx_muxer_t::execute(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > host_command = "", int argc = 0n1, wchar_t ** argv = 0x02b54b98, struct host_startup_info_t * host_info = 0x0285f8f4, wchar_t * result_buffer = 0x00000000 "", int buffer_size = 0n0, int * required_buffer_size = 0x00000000)+0x225 [d:\workspace\_work\1\s\src\corehost\cli\fxr\fx_muxer.cpp @ 541] 
64 0285f940 006bfb5e hostfxr!hostfxr_main_startupinfo(int argc = 0n1, wchar_t ** argv = 0x02b54b98, wchar_t * host_path = 0x02b65ee8 
65 0285fa90 006bfdae Lindexi_exe!exe_start(int argc = 0n1, wchar_t ** argv = 0x02b54b98)+0x7ce [d:\workspace\_work\1\s\src\corehost\corehost.cpp @ 220] 
66 0285faac 006c15fc Lindexi_exe!wmain(int argc = 0n1, wchar_t ** argv = 0x02b54b98)+0x6e [d:\workspace\_work\1\s\src\corehost\corehost.cpp @ 287] 
67 (Inline) -------- Lindexi_exe!invoke_main(void)+0x1c [d:\agent\_work\3\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl @ 90] 
68 0285faf4 7670fa29 Lindexi_exe!__scrt_common_main_seh(void)+0xfa [d:\agent\_work\3\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl @ 288] 
69 0285fb04 77ae7c3e kernel32!BaseThreadInitThunk+0x19
6a 0285fb60 77ae7c0e ntdll!__RtlUserThreadStart+0x2f
6b 0285fb70 00000000 ntdll!_RtlUserThreadStart+0x1b
```

以上的堆栈是我将 DUMP 文件放入到 VisualStudio 开启混合调试，加载符号之后看到的

卡顿的原因是主线程在获取嵌入的 UWP 控件的 UIA 相关信息时需要通过 COM 调用方式，但是 COM 调用没有返回

但我没有调查到更具体的 COM 调用没有返回的原因


以上的堆栈是在主线程收到了 `WM_GETOBJECT` 消息才会去获取嵌入的 UWP 控件的 UIA 相关信息，而 `WM_GETOBJECT` 消息就是 UIA 的消息之一，通过上面堆栈的以下两行可以知道是这样的原因

```
3a 0285e7f4 082efd67 PresentationCore!System.Windows.Interop.HwndTarget.CriticalHandleWMGetobject+0x56
3b 0285e8a8 082eb1fb PresentationCore!System.Windows.Interop.HwndTarget.HandleMessage+0x5d7
```

通过阅读 WPF 的源代码可以看到，进入 CriticalHandleWMGetobject 方法的路径，只有收到 `WM_GETOBJECT` 消息才能进来。通过 VisualStudio 看局部变量，也能看到当前的消息是 `WM_GETOBJECT` 消息

进入到 CriticalHandleWMGetobject 方法之后，将会调用 UIA 模块的 [AutomationInteropProvider.HostProviderFromHandle](https://learn.microsoft.com/en-us/dotnet/api/system.windows.automation.provider.automationinteropprovider.hostproviderfromhandle?view=windowsdesktop-7.0) 方法，在 UIA 模块的这个方法将会通过 COM 的方式继续调用

导致主线程卡住的原因就是 UIA 模块的 [AutomationInteropProvider.HostProviderFromHandle](https://learn.microsoft.com/en-us/dotnet/api/system.windows.automation.provider.automationinteropprovider.hostproviderfromhandle?view=windowsdesktop-7.0) 方法没有返回

通过调用堆栈的局部变量可以看到此时获取的是 UWP 控件的 UIA 信息

但是看 UWP 的主线程却没有卡住，只是在等渲染线程返回

调试 UWP 的渲染线程过于复杂，我就没有继续调试，暂不知道根本原因

主线程的堆栈上方是进入 PostMessageW 方法，这个方法不是同步发送消息，换句话说不是因为发送消息而卡住。卡住的地方是在 CCliModalLoop::MyDispatchMessage 之类的方法没有返回


这个问题也有可能是客户的机器存在一个奇怪的 UIA 进程，发送了错误的 UIA 消息，或者是执行 UIA 处理时不符合规范

规避方法是：

吃掉 `WM_GETOBJECT` 消息


实现方法是通过消息钩子吃掉自己的 `WM_GETOBJECT` 消息，代码如下

```csharp
    public MainWindow()
    {
        InitializeComponent();

        SourceInitialized += MainWindow_SourceInitialized;
    }

    private void MainWindow_SourceInitialized(object? sender, EventArgs e)
    {
        var windowInteropHelper = new WindowInteropHelper(this);
        var hwnd = windowInteropHelper.Handle;
        HookUIAutomationMessage(hwnd);
    }

    private void HookUIAutomationMessage(IntPtr windowPtr)
    {
        // 获取原本的消息处理
        var hWnd = new HWND(windowPtr);
        _oldWndProc = (IntPtr) Windows.Win32.PInvoke.GetWindowLong(hWnd, WINDOW_LONG_PTR_INDEX.GWL_WNDPROC);

        // 加入新的过滤处理
        WindowProc? hookUIAutomationMessageWndProc = HookUIAutomationMessageWndProc;
        _hookUIAutomationMessageWndProcPointer = Marshal.GetFunctionPointerForDelegate(hookUIAutomationMessageWndProc);
        Windows.Win32.PInvoke.SetWindowLong(hWnd, WINDOW_LONG_PTR_INDEX.GWL_WNDPROC,
            // 这里强转 int 只有 x86 下才可用
            (int) _hookUIAutomationMessageWndProcPointer);
    }

    private IntPtr HookUIAutomationMessageWndProc(IntPtr hwnd, uint msg, IntPtr wparam, IntPtr lparam)
    {
        const uint WM_GETOBJECT = 0x003D;
        if (msg == WM_GETOBJECT)
        {
            // 吃掉消息，防止主线程卡住
            return IntPtr.Zero;
        }

        // 过滤通过之后，再调用原先的消息循环
        return CallWindowProc(_oldWndProc, hwnd, msg, wparam, lparam);
    }

    public delegate IntPtr WindowProc(IntPtr hwnd, uint msg, IntPtr wParam, IntPtr lParam);

    private IntPtr _oldWndProc;
    private IntPtr _hookUIAutomationMessageWndProcPointer;

    [DllImport("user32", CharSet = CharSet.Unicode)]
    public static extern IntPtr CallWindowProc(IntPtr lpPrevWndFunc, IntPtr hWnd, uint uMsg, IntPtr wParam,
        IntPtr lParam);
```

以上规避方法存在的缺点是会让 UIA 功能失效，也就是将会丢失 UI 自动化测试功能、丢失残障辅助功能等

以上代码依赖 [CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html )

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/5c735d85bc02507f27f2270029050e402b580810/CucherelahiBewilargalkalbea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5c735d85bc02507f27f2270029050e402b580810/CucherelahiBewilargalkalbea) 上，可以通过以下方式获取整个项目的代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5c735d85bc02507f27f2270029050e402b580810
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5c735d85bc02507f27f2270029050e402b580810
```

获取代码之后，进入 CucherelahiBewilargalkalbea 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
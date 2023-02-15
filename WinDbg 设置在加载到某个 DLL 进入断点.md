# WinDbg 设置在加载到某个 DLL 进入断点

本文记录如何在 WinDbg 里，设置在加载到某个 DLL 时，自动进入断点。通过此方式用来定位是哪个业务模块加载了某个 DLL 模块

<!--more-->
<!-- CreateTime:2023/2/13 20:10:03 -->

<!-- 发布 -->
<!-- 博客 -->

在 WinDbg 里面，可以附加到现有进程，也可以启动某个进程。本文这里需要的是调试应用启动过程，是在哪个过程加载了某个指定的 DLL 库，于是就使用 Ctrl+E 快捷键，在 WinDbg 里面选择我需要调试的应用的 exe 文件进行启动

在 WinDbg 启动的进程默认将会进入暂停，方便输入命令

在 WinDbg 里设置在加载到某个 DLL 进入断点，可以使用如下命令

```
sxe ld:xxx.dll
```

将 `xxx.dll` 替换为需要关注的 DLL 名即可。如果有多个 DLL 都需要关注，那就输入多次，每次使用不同的 DLL 名

输入完成之后，输入 g 让 WinDbg 继续执行进程

等待进程加载到 `xxx.dll` 时，将会自动进入断点。此时大部分情况下就可以通过输入 `kp` 命令查看调用堆栈，通过调用堆栈了解到是哪个业务加载到了 DLL 库

例如调试某个 WPF 应用在启动过程哪个业务模块开始加载 PresentationCore.dll 库

先使用 Ctrl+E 快捷键，在 WinDbg 选择需要启动的应用。启动完成之后，输入 `sxe ld:PresentationCore.dll` 设置断点，接着按下 `g` 继续执行，可以看到 WinDbg 输出大概如下

```
************* Path validation summary **************
Response                         Time (ms)     Location
Deferred                                       srv*
Symbol search path is: srv*
Executable search path is: 
ModLoad: 00007ff6`a6f50000 00007ff6`a6f7a000   apphost.exe
ModLoad: 00007ffe`5fe10000 00007ffe`60008000   ntdll.dll
ModLoad: 00007ffe`5eed0000 00007ffe`5ef8f000   C:\Windows\System32\KERNEL32.DLL
ModLoad: 00007ffe`5d810000 00007ffe`5dae2000   C:\Windows\System32\KERNELBASE.dll
ModLoad: 00007ffe`5e730000 00007ffe`5e8d1000   C:\Windows\System32\USER32.dll
ModLoad: 00007ffe`5d750000 00007ffe`5d772000   C:\Windows\System32\win32u.dll
ModLoad: 00007ffe`5f160000 00007ffe`5f18b000   C:\Windows\System32\GDI32.dll
ModLoad: 00007ffe`5dd40000 00007ffe`5de4f000   C:\Windows\System32\gdi32full.dll
ModLoad: 00007ffe`5d600000 00007ffe`5d69d000   C:\Windows\System32\msvcp_win.dll
ModLoad: 00007ffe`5d500000 00007ffe`5d600000   C:\Windows\System32\ucrtbase.dll
ModLoad: 00007ffe`5f260000 00007ffe`5f9a4000   C:\Windows\System32\SHELL32.dll
ModLoad: 00007ffe`5e8e0000 00007ffe`5e98e000   C:\Windows\System32\ADVAPI32.dll
ModLoad: 00007ffe`5e110000 00007ffe`5e1ae000   C:\Windows\System32\msvcrt.dll
ModLoad: 00007ffe`5f0b0000 00007ffe`5f14c000   C:\Windows\System32\sechost.dll
ModLoad: 00007ffe`5de50000 00007ffe`5df75000   C:\Windows\System32\RPCRT4.dll
(5918.9f6c): Break instruction exception - code 80000003 (first chance)
ntdll!LdrpDoDebuggerBreak+0x30:
00007ffe`5fee0950 cc              int     3
0:000> sxe ld:PresentationCore.dll
0:000> g
```

应用加载 `PresentationCore.dll` 进入断点，输入 `kp` 可以看到 WinDbg 输出大概如下

```
0:000> kp
 # Child-SP          RetAddr           Call Site
00 00000072`d6779a88 00007ffe`5fe24d42 ntdll!NtMapViewOfSection+0x14
01 00000072`d6779a90 00007ffe`5fe24aaa ntdll!LdrpMinimalMapModule+0x10a
02 00000072`d6779b50 00007ffe`5fe6fd83 ntdll!LdrpMapDllWithSectionHandle+0x1a
03 00000072`d6779ba0 00007ffe`5fe6fab0 ntdll!LdrpMapDllNtFileName+0x19f
04 00000072`d6779ca0 00007ffe`5fe6ed4f ntdll!LdrpMapDllFullPath+0xe0
05 00000072`d6779e30 00007ffe`5fe2fb53 ntdll!LdrpProcessWork+0x123
06 00000072`d6779e90 00007ffe`5fe273e4 ntdll!LdrpLoadDllInternal+0x13f
07 00000072`d6779f10 00007ffe`5fe26af4 ntdll!LdrpLoadDll+0xa8
08 00000072`d677a0c0 00007ffe`5d8454c2 ntdll!LdrLoadDll+0xe4
09 00000072`d677a1b0 00007ffe`34cc738f KERNELBASE!LoadLibraryExW+0x162
0a 00000072`d677a220 00007ffe`34cc71dc coreclr!LoadLibraryExWrapper(wchar_t * lpLibFileName = 0x00000072`00008000 "--- memory read error at address 0x00000072`00008000 ---", unsigned long dwFlags = 8)+0x153 [D:\a\_work\1\s\src\coreclr\utilcode\longfilepathwrappers.cpp @ 68] 
0b (Inline Function) --------`-------- coreclr!CLRLoadLibraryExWorker(void)+0x21 [D:\a\_work\1\s\src\coreclr\vm\util.cpp @ 1269] 
0c (Inline Function) --------`-------- coreclr!CLRLoadLibraryEx(void)+0x21 [D:\a\_work\1\s\src\coreclr\vm\util.cpp @ 1288] 
0d 00000072`d677a4f0 00007ffe`34cc70db coreclr!LoadedImageLayout::LoadedImageLayout(class PEImage * pOwner = 0x00000208`7cb566a0, int bNTSafeLoad = <Value unavailable error>, HRESULT * returnDontThrow = 0x00000072`d677a5f0)+0xa0 [D:\a\_work\1\s\src\coreclr\vm\peimagelayout.cpp @ 696] 
0e 00000072`d677a540 00007ffe`34cc7043 coreclr!PEImageLayout::Load(class PEImage * pOwner = 0x00000208`7cb566a0, int bNTSafeLoad = 0n0, HRESULT * returnDontThrow = 0x00000072`d677a5f0)+0x63 [D:\a\_work\1\s\src\coreclr\vm\peimagelayout.cpp @ 72] 
0f 00000072`d677a580 00007ffe`34ccb80e coreclr!PEImage::CreateLayoutMapped(void)+0x67 [D:\a\_work\1\s\src\coreclr\vm\peimage.cpp @ 1037] 
10 00000072`d677a5f0 00007ffe`34ccb739 coreclr!PEImage::GetLayoutInternal(unsigned long imageLayoutMask = <Value unavailable error>, unsigned long flags = <Value unavailable error>)+0x5e [D:\a\_work\1\s\src\coreclr\vm\peimage.cpp @ 1004] 
11 00000072`d677a620 00007ffe`34c6710e coreclr!PEImage::GetLayout(unsigned long imageLayoutMask = 0xf, unsigned long flags = 1)+0x111 [D:\a\_work\1\s\src\coreclr\vm\peimage.cpp @ 939] 
12 00000072`d677a6a0 00007ffe`34cc7914 coreclr!BinderAcquireImport(class PEImage * pPEImage = 0x00000208`7cb566a0, struct IMDInternalImport ** ppIAssemblyMetaDataImport = 0x00000072`d677a7c0, unsigned long * pdwPAFlags = 0x00000072`d677a7f8, int bNativeImage = 0n0)+0x96 [D:\a\_work\1\s\src\coreclr\vm\coreassemblyspec.cpp @ 257] 
13 00000072`d677a780 00007ffe`34cc6757 coreclr!BINDER_SPACE::AssemblyBinder::GetAssembly(class SString * assemblyPath = 0x00000072`d677a930 [Unicode] "C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App\6.0.13\PresentationCore.dll", int fIsInGAC = 0n1, class BINDER_SPACE::Assembly ** ppAssembly = 0x00000072`d677a8c8, struct BundleFileLocation * bundleFileLocation = 0x00000072`d677a9c0)+0x134 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 1253] 
14 00000072`d677a890 00007ffe`34cca7fa coreclr!BINDER_SPACE::AssemblyBinder::BindByTpaList(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pRequestedAssemblyName = 0x00000208`7cb4ff70, bool excludeAppPaths = false, class BINDER_SPACE::BindResult * pBindResult = 0x00000072`d677ad90)+0x123 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 1101] 
15 00000072`d677ab40 00007ffe`34cca687 coreclr!BINDER_SPACE::AssemblyBinder::BindLocked(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, bool skipVersionCompatibilityCheck = false, bool excludeAppPaths = false, class BINDER_SPACE::BindResult * pBindResult = 0x00000072`d677ad90)+0x82 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 687] 
16 00000072`d677ab90 00007ffe`34cc97d1 coreclr!BINDER_SPACE::AssemblyBinder::BindByName(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, bool skipFailureCaching = false, bool skipVersionCompatibilityCheck = false, bool excludeAppPaths = false, class BINDER_SPACE::BindResult * pBindResult = 0x00000072`d677ad90)+0xd7 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 535] 
17 00000072`d677ad30 00007ffe`34cc9270 coreclr!BINDER_SPACE::AssemblyBinder::BindAssembly(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, wchar_t * szCodeBase = 0x00000000`00000000 "", bool excludeAppPaths = false, class BINDER_SPACE::Assembly ** ppAssembly = 0x00000072`d677b1b8)+0xdd [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 279] 
18 00000072`d677b130 00007ffe`34cc907a coreclr!CLRPrivBinderCoreCLR::BindAssemblyByNameWorker(class BINDER_SPACE::AssemblyName * pAssemblyName = <Value unavailable error>, class BINDER_SPACE::Assembly ** ppCoreCLRFoundAssembly = 0x00000072`d677b1b8, bool excludeAppPaths = <Value unavailable error>)+0x30 [D:\a\_work\1\s\src\coreclr\binder\clrprivbindercoreclr.cpp @ 34] 
19 00000072`d677b180 00007ffe`34cc8fcd coreclr!CLRPrivBinderCoreCLR::BindUsingAssemblyName(class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, struct ICLRPrivAssembly ** ppAssembly = 0x00000072`d677b2c0)+0x76 [D:\a\_work\1\s\src\coreclr\binder\clrprivbindercoreclr.cpp @ 73] 
1a 00000072`d677b250 00007ffe`34ccabdd coreclr!CLRPrivBinderCoreCLR::BindAssemblyByName(struct AssemblyNameData * pAssemblyNameData = <Value unavailable error>, struct ICLRPrivAssembly ** ppAssembly = 0x00000072`d677b2c0)+0xad [D:\a\_work\1\s\src\coreclr\binder\clrprivbindercoreclr.cpp @ 57] 
1b 00000072`d677b290 00007ffe`34cc9ec3 coreclr!AssemblySpec::Bind(class AppDomain * pAppDomain = <Value unavailable error>, struct CoreBindResult * pResult = 0x00000072`d677bca8)+0x131 [D:\a\_work\1\s\src\coreclr\vm\coreassemblyspec.cpp @ 133] 
1c 00000072`d677bbd0 00007ffe`34cafe26 coreclr!AppDomain::BindAssemblySpec(class AssemblySpec * pSpec = 0x00000072`d677c350, int fThrowOnFileNotFound = 0n1)+0x193 [D:\a\_work\1\s\src\coreclr\vm\appdomain.cpp @ 3984] 
1d 00000072`d677c320 00007ffe`34c821c8 coreclr!PEFile::LoadAssembly(unsigned int kAssemblyRef = 0x23000007)+0xbe [D:\a\_work\1\s\src\coreclr\vm\pefile.cpp @ 1479] 
1e 00000072`d677c3d0 00007ffe`34d6c192 coreclr!Module::LoadAssembly(unsigned int kAssemblyRef = 0x23000007)+0x10c [D:\a\_work\1\s\src\coreclr\vm\ceeload.cpp @ 4679] 
1f 00000072`d677c4e0 00007ffe`34cbc859 coreclr!Assembly::FindModuleByTypeRef(class Module * pModule = 0x00007ffd`d53083a8, unsigned int tkType = 0x23000007, Loader::LoadFlag loadFlag = Load (0n0), int * pfNoResolutionScope = 0x00000000`00000007)+0x332 [D:\a\_work\1\s\src\coreclr\vm\assembly.cpp @ 1083] 
20 00000072`d677c600 00007ffe`34c71f82 coreclr!ClassLoader::LoadTypeDefOrRefThrowing(class Module * pModule = 0x00007ffd`d53083a8, unsigned int typeDefOrRef = 0x10004e4, ClassLoader::NotFoundAction fNotFoundAction = ThrowIfNotFound (0n0), ClassLoader::PermitUninstantiatedFlag fUninstantiated = PermitUninstDefOrRef (0n1), unsigned int tokenNotToLoad = 0, ClassLoadLevel level = CLASS_LOADED (0n6))+0x289 [D:\a\_work\1\s\src\coreclr\vm\clsload.cpp @ 2612] 
21 00000072`d677c6f0 00007ffe`34d2003b coreclr!MemberLoader::GetDescFromMemberRef(class Module * pModule = 0x00007ffd`d53083a8, unsigned int MemberRef = 0xa000c6b, class MethodDesc ** ppMD = 0x00000072`d677c9f8, class FieldDesc ** ppFD = 0x00000072`d677cc30, class SigTypeContext * pTypeContext = 0x00000072`d677cd38, int strictMetadataChecks = 0n0, class TypeHandle * ppTH = 0x00000072`d677ca30, int actualTypeRequired = 0n0, unsigned char ** ppTypeSig = 0x00000000`00000000, unsigned long * pcbTypeSig = 0x00000000`00000000)+0x332 [D:\a\_work\1\s\src\coreclr\vm\memberload.cpp @ 295] 
22 00000072`d677c9a0 00007ffe`34dbc0c1 coreclr!ExternalMethodFixupWorker(struct TransitionBlock * pTransitionBlock = <Value unavailable error>, unsigned int64 pIndirection = 0x00007ffe`33ff6550, unsigned long sectionIndex = 0x1c, class Module * pModule = 0x00007ffd`d53083a8)+0xe4b [D:\a\_work\1\s\src\coreclr\vm\prestub.cpp @ 2659] 
23 00000072`d677d120 00007ffe`3343266b coreclr!DelayLoad_MethodCall+0x71
24 00000072`d677d1d0 00007ffe`3342ebf6 PresentationFramework!System.Windows.Application::ApplicationInit+0x3b
25 00000072`d677d210 00007ffe`34dbae93 PresentationFramework!System.Windows.Application::.cctor+0x66
26 00000072`d677d250 00007ffe`34cb6dd0 coreclr!CallDescrWorkerInternal+0x83
27 00000072`d677d290 00007ffe`34cb6d7c coreclr!DispatchCallDebuggerWrapper(struct CallDescrData * pCallDescrData = 0x00007ffe`33d18f5c, int fCriticalCall = 0n870126915)+0x1c [D:\a\_work\1\s\src\coreclr\vm\callhelpers.cpp @ 158] 
28 00000072`d677d2f0 00007ffe`34cb6c60 coreclr!DispatchCallSimple(unsigned int64 * pSrc = 0x00000072`d677d478, unsigned long numStackSlotsToCopy = 4, unsigned int64 pTargetAddress = 0x00007ffd`d523fb50, unsigned long dwDispatchCallSimpleFlags = 2)+0x60 [D:\a\_work\1\s\src\coreclr\vm\callhelpers.cpp @ 223] 
29 00000072`d677d380 00007ffe`34cb7b81 coreclr!MethodTable::RunClassInitEx(class Object ** pThrowable = 0x00000072`d677d528)+0x148 [D:\a\_work\1\s\src\coreclr\vm\methodtable.cpp @ 3184] 
2a 00000072`d677d4c0 00007ffe`34cb7dd8 coreclr!MethodTable::DoRunClassInitThrowing(void)+0x2ed [D:\a\_work\1\s\src\coreclr\vm\methodtable.cpp @ 3369] 
2b 00000072`d677df10 00007ffe`34d3cf2b coreclr!MethodTable::CheckRunClassInitThrowing(void)+0xd8 [D:\a\_work\1\s\src\coreclr\vm\methodtable.cpp @ 3510] 
2c 00000072`d677df40 00007ffe`34d3ca08 coreclr!DynamicHelperFixup(struct TransitionBlock * pTransitionBlock = 0x00000072`d677e508, unsigned int64 * pCell = 0x00007ffe`33fe0720, unsigned long sectionIndex = 0, class Module * pModule = 0x00007ffd`d53083a8, CORCOMPILE_FIXUP_BLOB_KIND * pKind = 0x00000072`d677e4b0, class TypeHandle * pTH = 0x00000072`d677e3a8, class MethodDesc ** ppMD = 0x00000072`d677e3c8, class FieldDesc ** ppFD = 0x00000072`d677e3c0)+0x427 [D:\a\_work\1\s\src\coreclr\vm\prestub.cpp @ 3276] 
2d 00000072`d677e360 00007ffe`34dbc18a coreclr!DynamicHelperWorker(struct TransitionBlock * pTransitionBlock = 0x00000072`d677e508, unsigned int64 * pCell = 0x00007ffe`33fe0720, unsigned long sectionIndex = 2, class Module * pModule = 0x00007ffd`d53083a8, int frameFlags = <Value unavailable error>)+0x168 [D:\a\_work\1\s\src\coreclr\vm\prestub.cpp @ 3585] 
2e 00000072`d677e490 00007ffe`3342ec20 coreclr!DelayLoad_Helper+0x7a
Unable to read dynamic function table entries
Unable to read dynamic function table entries
2f 00000072`d677e550 00007ffd`d5242e35 PresentationFramework!System.Windows.Application::.ctor+0x20
Unable to read dynamic function table entries
Unable to read dynamic function table entries
Unable to read dynamic function table entries
30 00000072`d677e5b0 00000208`0000dc70 0x00007ffd`d5242e35
Unable to read dynamic function table entries
31 00000072`d677e5b8 00000208`0000dd40 0x00000208`0000dc70
32 00000072`d677e5c0 00007ffe`34d5f3fb 0x00000208`0000dd40
33 (Inline Function) --------`-------- coreclr!MetaSig::GetReturnType(void)+0x14 [D:\a\_work\1\s\src\coreclr\vm\siginfo.cpp @ 5243] 
34 (Inline Function) --------`-------- coreclr!MetaSig::IsReturnTypeVoid(void)+0x14 [D:\a\_work\1\s\src\coreclr\vm\siginfo.cpp @ 5249] 
35 00000072`d677e5c8 00007ffd`d53be898 coreclr!MethodDesc::IsVoid(void)+0x2f [D:\a\_work\1\s\src\coreclr\vm\method.cpp @ 1071] 
36 00000072`d677e678 00000072`d677e878 0x00007ffd`d53be898
37 00000072`d677e680 00000000`00000020 0x00000072`d677e878
38 00000072`d677e688 00000072`d677e808 0x20
39 00000072`d677e690 00000072`d677e6b0 0x00000072`d677e808
3a 00000072`d677e698 00007ffe`00000001 0x00000072`d677e6b0
3b 00000072`d677e6a0 00000000`00000000 0x00007ffe`00000001
```

这里面有一些缺乏符号的，通过 [WinDbg 加载 dotnet core 的 sos.dll 辅助调试方法](https://blog.lindexi.com/post/WinDbg-%E5%8A%A0%E8%BD%BD-dotnet-core-%E7%9A%84-sos.dll-%E8%BE%85%E5%8A%A9%E8%B0%83%E8%AF%95%E6%96%B9%E6%B3%95.html ) 提供的方法，在安装好工具之后，加载 sos.dll 之后再次输入 `kp` 即可拿到堆栈详细信息

```
0:000> .load C:\Users\linde\.dotnet\sos\sos.dll
0:000> kp
 # Child-SP          RetAddr           Call Site
00 00000072`d6779a88 00007ffe`5fe24d42 ntdll!NtMapViewOfSection+0x14
01 00000072`d6779a90 00007ffe`5fe24aaa ntdll!LdrpMinimalMapModule+0x10a
02 00000072`d6779b50 00007ffe`5fe6fd83 ntdll!LdrpMapDllWithSectionHandle+0x1a
03 00000072`d6779ba0 00007ffe`5fe6fab0 ntdll!LdrpMapDllNtFileName+0x19f
04 00000072`d6779ca0 00007ffe`5fe6ed4f ntdll!LdrpMapDllFullPath+0xe0
05 00000072`d6779e30 00007ffe`5fe2fb53 ntdll!LdrpProcessWork+0x123
06 00000072`d6779e90 00007ffe`5fe273e4 ntdll!LdrpLoadDllInternal+0x13f
07 00000072`d6779f10 00007ffe`5fe26af4 ntdll!LdrpLoadDll+0xa8
08 00000072`d677a0c0 00007ffe`5d8454c2 ntdll!LdrLoadDll+0xe4
09 00000072`d677a1b0 00007ffe`34cc738f KERNELBASE!LoadLibraryExW+0x162
0a 00000072`d677a220 00007ffe`34cc71dc coreclr!LoadLibraryExWrapper(wchar_t * lpLibFileName = 0x00000072`00008000 "--- memory read error at address 0x00000072`00008000 ---", unsigned long dwFlags = 8)+0x153 [D:\a\_work\1\s\src\coreclr\utilcode\longfilepathwrappers.cpp @ 68] 
0b (Inline Function) --------`-------- coreclr!CLRLoadLibraryExWorker(void)+0x21 [D:\a\_work\1\s\src\coreclr\vm\util.cpp @ 1269] 
0c (Inline Function) --------`-------- coreclr!CLRLoadLibraryEx(void)+0x21 [D:\a\_work\1\s\src\coreclr\vm\util.cpp @ 1288] 
0d 00000072`d677a4f0 00007ffe`34cc70db coreclr!LoadedImageLayout::LoadedImageLayout(class PEImage * pOwner = 0x00000208`7cb566a0, int bNTSafeLoad = <Value unavailable error>, HRESULT * returnDontThrow = 0x00000072`d677a5f0)+0xa0 [D:\a\_work\1\s\src\coreclr\vm\peimagelayout.cpp @ 696] 
0e 00000072`d677a540 00007ffe`34cc7043 coreclr!PEImageLayout::Load(class PEImage * pOwner = 0x00000208`7cb566a0, int bNTSafeLoad = 0n0, HRESULT * returnDontThrow = 0x00000072`d677a5f0)+0x63 [D:\a\_work\1\s\src\coreclr\vm\peimagelayout.cpp @ 72] 
0f 00000072`d677a580 00007ffe`34ccb80e coreclr!PEImage::CreateLayoutMapped(void)+0x67 [D:\a\_work\1\s\src\coreclr\vm\peimage.cpp @ 1037] 
10 00000072`d677a5f0 00007ffe`34ccb739 coreclr!PEImage::GetLayoutInternal(unsigned long imageLayoutMask = <Value unavailable error>, unsigned long flags = <Value unavailable error>)+0x5e [D:\a\_work\1\s\src\coreclr\vm\peimage.cpp @ 1004] 
11 00000072`d677a620 00007ffe`34c6710e coreclr!PEImage::GetLayout(unsigned long imageLayoutMask = 0xf, unsigned long flags = 1)+0x111 [D:\a\_work\1\s\src\coreclr\vm\peimage.cpp @ 939] 
12 00000072`d677a6a0 00007ffe`34cc7914 coreclr!BinderAcquireImport(class PEImage * pPEImage = 0x00000208`7cb566a0, struct IMDInternalImport ** ppIAssemblyMetaDataImport = 0x00000072`d677a7c0, unsigned long * pdwPAFlags = 0x00000072`d677a7f8, int bNativeImage = 0n0)+0x96 [D:\a\_work\1\s\src\coreclr\vm\coreassemblyspec.cpp @ 257] 
13 00000072`d677a780 00007ffe`34cc6757 coreclr!BINDER_SPACE::AssemblyBinder::GetAssembly(class SString * assemblyPath = 0x00000072`d677a930 [Unicode] "C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App\6.0.13\PresentationCore.dll", int fIsInGAC = 0n1, class BINDER_SPACE::Assembly ** ppAssembly = 0x00000072`d677a8c8, struct BundleFileLocation * bundleFileLocation = 0x00000072`d677a9c0)+0x134 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 1253] 
14 00000072`d677a890 00007ffe`34cca7fa coreclr!BINDER_SPACE::AssemblyBinder::BindByTpaList(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pRequestedAssemblyName = 0x00000208`7cb4ff70, bool excludeAppPaths = false, class BINDER_SPACE::BindResult * pBindResult = 0x00000072`d677ad90)+0x123 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 1101] 
15 00000072`d677ab40 00007ffe`34cca687 coreclr!BINDER_SPACE::AssemblyBinder::BindLocked(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, bool skipVersionCompatibilityCheck = false, bool excludeAppPaths = false, class BINDER_SPACE::BindResult * pBindResult = 0x00000072`d677ad90)+0x82 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 687] 
16 00000072`d677ab90 00007ffe`34cc97d1 coreclr!BINDER_SPACE::AssemblyBinder::BindByName(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, bool skipFailureCaching = false, bool skipVersionCompatibilityCheck = false, bool excludeAppPaths = false, class BINDER_SPACE::BindResult * pBindResult = 0x00000072`d677ad90)+0xd7 [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 535] 
17 00000072`d677ad30 00007ffe`34cc9270 coreclr!BINDER_SPACE::AssemblyBinder::BindAssembly(class BINDER_SPACE::ApplicationContext * pApplicationContext = 0x00000208`7cacebd8, class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, wchar_t * szCodeBase = 0x00000000`00000000 "", bool excludeAppPaths = false, class BINDER_SPACE::Assembly ** ppAssembly = 0x00000072`d677b1b8)+0xdd [D:\a\_work\1\s\src\coreclr\binder\assemblybinder.cpp @ 279] 
18 00000072`d677b130 00007ffe`34cc907a coreclr!CLRPrivBinderCoreCLR::BindAssemblyByNameWorker(class BINDER_SPACE::AssemblyName * pAssemblyName = <Value unavailable error>, class BINDER_SPACE::Assembly ** ppCoreCLRFoundAssembly = 0x00000072`d677b1b8, bool excludeAppPaths = <Value unavailable error>)+0x30 [D:\a\_work\1\s\src\coreclr\binder\clrprivbindercoreclr.cpp @ 34] 
19 00000072`d677b180 00007ffe`34cc8fcd coreclr!CLRPrivBinderCoreCLR::BindUsingAssemblyName(class BINDER_SPACE::AssemblyName * pAssemblyName = 0x00000208`7cb4ff70, struct ICLRPrivAssembly ** ppAssembly = 0x00000072`d677b2c0)+0x76 [D:\a\_work\1\s\src\coreclr\binder\clrprivbindercoreclr.cpp @ 73] 
1a 00000072`d677b250 00007ffe`34ccabdd coreclr!CLRPrivBinderCoreCLR::BindAssemblyByName(struct AssemblyNameData * pAssemblyNameData = <Value unavailable error>, struct ICLRPrivAssembly ** ppAssembly = 0x00000072`d677b2c0)+0xad [D:\a\_work\1\s\src\coreclr\binder\clrprivbindercoreclr.cpp @ 57] 
1b 00000072`d677b290 00007ffe`34cc9ec3 coreclr!AssemblySpec::Bind(class AppDomain * pAppDomain = <Value unavailable error>, struct CoreBindResult * pResult = 0x00000072`d677bca8)+0x131 [D:\a\_work\1\s\src\coreclr\vm\coreassemblyspec.cpp @ 133] 
1c 00000072`d677bbd0 00007ffe`34cafe26 coreclr!AppDomain::BindAssemblySpec(class AssemblySpec * pSpec = 0x00000072`d677c350, int fThrowOnFileNotFound = 0n1)+0x193 [D:\a\_work\1\s\src\coreclr\vm\appdomain.cpp @ 3984] 
1d 00000072`d677c320 00007ffe`34c821c8 coreclr!PEFile::LoadAssembly(unsigned int kAssemblyRef = 0x23000007)+0xbe [D:\a\_work\1\s\src\coreclr\vm\pefile.cpp @ 1479] 
1e 00000072`d677c3d0 00007ffe`34d6c192 coreclr!Module::LoadAssembly(unsigned int kAssemblyRef = 0x23000007)+0x10c [D:\a\_work\1\s\src\coreclr\vm\ceeload.cpp @ 4679] 
1f 00000072`d677c4e0 00007ffe`34cbc859 coreclr!Assembly::FindModuleByTypeRef(class Module * pModule = 0x00007ffd`d53083a8, unsigned int tkType = 0x23000007, Loader::LoadFlag loadFlag = Load (0n0), int * pfNoResolutionScope = 0x00000000`00000007)+0x332 [D:\a\_work\1\s\src\coreclr\vm\assembly.cpp @ 1083] 
20 00000072`d677c600 00007ffe`34c71f82 coreclr!ClassLoader::LoadTypeDefOrRefThrowing(class Module * pModule = 0x00007ffd`d53083a8, unsigned int typeDefOrRef = 0x10004e4, ClassLoader::NotFoundAction fNotFoundAction = ThrowIfNotFound (0n0), ClassLoader::PermitUninstantiatedFlag fUninstantiated = PermitUninstDefOrRef (0n1), unsigned int tokenNotToLoad = 0, ClassLoadLevel level = CLASS_LOADED (0n6))+0x289 [D:\a\_work\1\s\src\coreclr\vm\clsload.cpp @ 2612] 
21 00000072`d677c6f0 00007ffe`34d2003b coreclr!MemberLoader::GetDescFromMemberRef(class Module * pModule = 0x00007ffd`d53083a8, unsigned int MemberRef = 0xa000c6b, class MethodDesc ** ppMD = 0x00000072`d677c9f8, class FieldDesc ** ppFD = 0x00000072`d677cc30, class SigTypeContext * pTypeContext = 0x00000072`d677cd38, int strictMetadataChecks = 0n0, class TypeHandle * ppTH = 0x00000072`d677ca30, int actualTypeRequired = 0n0, unsigned char ** ppTypeSig = 0x00000000`00000000, unsigned long * pcbTypeSig = 0x00000000`00000000)+0x332 [D:\a\_work\1\s\src\coreclr\vm\memberload.cpp @ 295] 
22 00000072`d677c9a0 00007ffe`34dbc0c1 coreclr!ExternalMethodFixupWorker(struct TransitionBlock * pTransitionBlock = <Value unavailable error>, unsigned int64 pIndirection = 0x00007ffe`33ff6550, unsigned long sectionIndex = 0x1c, class Module * pModule = 0x00007ffd`d53083a8)+0xe4b [D:\a\_work\1\s\src\coreclr\vm\prestub.cpp @ 2659] 
23 00000072`d677d120 00007ffe`3343266b coreclr!DelayLoad_MethodCall+0x71
24 00000072`d677d1d0 00007ffe`3342ebf6 PresentationFramework!System.Windows.Application::ApplicationInit+0x3b
25 00000072`d677d210 00007ffe`34dbae93 PresentationFramework!System.Windows.Application::.cctor+0x66
26 00000072`d677d250 00007ffe`34cb6dd0 coreclr!CallDescrWorkerInternal+0x83
27 00000072`d677d290 00007ffe`34cb6d7c coreclr!DispatchCallDebuggerWrapper(struct CallDescrData * pCallDescrData = 0x00007ffe`33d18f5c, int fCriticalCall = 0n870126915)+0x1c [D:\a\_work\1\s\src\coreclr\vm\callhelpers.cpp @ 158] 
28 00000072`d677d2f0 00007ffe`34cb6c60 coreclr!DispatchCallSimple(unsigned int64 * pSrc = 0x00000072`d677d478, unsigned long numStackSlotsToCopy = 4, unsigned int64 pTargetAddress = 0x00007ffd`d523fb50, unsigned long dwDispatchCallSimpleFlags = 2)+0x60 [D:\a\_work\1\s\src\coreclr\vm\callhelpers.cpp @ 223] 
29 00000072`d677d380 00007ffe`34cb7b81 coreclr!MethodTable::RunClassInitEx(class Object ** pThrowable = 0x00000072`d677d528)+0x148 [D:\a\_work\1\s\src\coreclr\vm\methodtable.cpp @ 3184] 
2a 00000072`d677d4c0 00007ffe`34cb7dd8 coreclr!MethodTable::DoRunClassInitThrowing(void)+0x2ed [D:\a\_work\1\s\src\coreclr\vm\methodtable.cpp @ 3369] 
2b 00000072`d677df10 00007ffe`34d3cf2b coreclr!MethodTable::CheckRunClassInitThrowing(void)+0xd8 [D:\a\_work\1\s\src\coreclr\vm\methodtable.cpp @ 3510] 
2c 00000072`d677df40 00007ffe`34d3ca08 coreclr!DynamicHelperFixup(struct TransitionBlock * pTransitionBlock = 0x00000072`d677e508, unsigned int64 * pCell = 0x00007ffe`33fe0720, unsigned long sectionIndex = 0, class Module * pModule = 0x00007ffd`d53083a8, CORCOMPILE_FIXUP_BLOB_KIND * pKind = 0x00000072`d677e4b0, class TypeHandle * pTH = 0x00000072`d677e3a8, class MethodDesc ** ppMD = 0x00000072`d677e3c8, class FieldDesc ** ppFD = 0x00000072`d677e3c0)+0x427 [D:\a\_work\1\s\src\coreclr\vm\prestub.cpp @ 3276] 
2d 00000072`d677e360 00007ffe`34dbc18a coreclr!DynamicHelperWorker(struct TransitionBlock * pTransitionBlock = 0x00000072`d677e508, unsigned int64 * pCell = 0x00007ffe`33fe0720, unsigned long sectionIndex = 2, class Module * pModule = 0x00007ffd`d53083a8, int frameFlags = <Value unavailable error>)+0x168 [D:\a\_work\1\s\src\coreclr\vm\prestub.cpp @ 3585] 
2e 00000072`d677e490 00007ffe`3342ec20 coreclr!DelayLoad_Helper+0x7a
2f 00000072`d677e550 00007ffd`d5242e35 PresentationFramework!System.Windows.Application::.ctor+0x20
30 00000072`d677e5b0 00007ffd`d5242dcf 0x00007ffd`d5242e35
31 00000072`d677e5e0 00007ffe`34dbae93 0x00007ffd`d5242dcf
32 00000072`d677e630 00007ffe`34cd83b6 coreclr!CallDescrWorkerInternal+0x83
33 00000072`d677e670 00007ffe`34d5f3c7 coreclr!MethodDescCallSite::CallTargetWorker(unsigned int64 * pArguments = 0x00000072`d677e8d8, unsigned int64 * pReturnValue = 0x00000000`00000000, int cbReturnValue = 0n0)+0x176 [D:\a\_work\1\s\src\coreclr\vm\callhelpers.cpp @ 551] 
34 (Inline Function) --------`-------- coreclr!MethodDescCallSite::Call(void)+0xb [D:\a\_work\1\s\src\coreclr\vm\callhelpers.h @ 458] 
35 00000072`d677e7a0 00007ffe`34d5f246 coreclr!RunMainInternal(struct Param * pParam = 0x00000072`d677e8f8)+0x11f [D:\a\_work\1\s\src\coreclr\vm\assembly.cpp @ 1483] 
36 00000072`d677e8d0 00007ffe`34d5f0f5 coreclr!RunMain(class MethodDesc * pFD = 0x00007ffd`d53be898, int * piRetVal = 0x00000072`d677e9c0, class PtrArray ** stringArgs = 0x00000072`d677ed38)+0xd2 [D:\a\_work\1\s\src\coreclr\vm\assembly.cpp @ 1554] 
37 00000072`d677e980 00007ffe`34d5eeb6 coreclr!Assembly::ExecuteMainMethod(class PtrArray ** stringArgs = 0x00000072`d677ed38)+0x1c9 [D:\a\_work\1\s\src\coreclr\vm\assembly.cpp @ 1672] 
38 00000072`d677ed10 00007ffe`34d761a2 coreclr!CorHost2::ExecuteAssembly(unsigned long dwAppDomainId = <Value unavailable error>, wchar_t * pwzAssemblyPath = 0x00000000`00000001 "--- memory read error at address 0x00000000`00000001 ---", int argc = 0n0, wchar_t ** argv = 0x00000000`00000000, unsigned long * pReturnValue = 0x00000072`d677ef60)+0x1c6 [D:\a\_work\1\s\src\coreclr\vm\corhost.cpp @ 384] 
39 00000072`d677ee80 00007ffe`3517971b coreclr!coreclr_execute_assembly(void * hostHandle = 0x00000208`7b1a00b0, unsigned int domainId = 1, int argc = 0n0, char ** argv = <Value unavailable error>, char * managedAssemblyPath = 0x00000208`7b12c560 "D:\???", unsigned int * exitCode = 0x00000072`d677ef60)+0xe2 [D:\a\_work\1\s\src\coreclr\dlls\mscoree\unixinterface.cpp @ 446] 
3a (Inline Function) --------`-------- hostpolicy!coreclr_t::execute_assembly(void)+0x2a [D:\a\_work\1\s\src\native\corehost\hostpolicy\coreclr.cpp @ 89] 
3b 00000072`d677ef20 00007ffe`35179a4c hostpolicy!run_app_for_context(struct hostpolicy_context_t * context = 0x00000208`7b12e500, int argc = 0n0, wchar_t ** argv = <Value unavailable error>)+0x56b [D:\a\_work\1\s\src\native\corehost\hostpolicy\hostpolicy.cpp @ 255] 
3c 00000072`d677f0c0 00007ffe`3517a3e7 hostpolicy!run_app(int argc = 0n0, wchar_t ** argv = 0x00000208`7b1187f8)+0x3c [D:\a\_work\1\s\src\native\corehost\hostpolicy\hostpolicy.cpp @ 284] 
3d 00000072`d677f100 00007ffe`351db77c hostpolicy!corehost_main(int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0)+0x107 [D:\a\_work\1\s\src\native\corehost\hostpolicy\hostpolicy.cpp @ 430] 
3e 00000072`d677f2b0 00007ffe`351de3f7 hostfxr!execute_app(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * impl_dll_dir = <Value unavailable error>, class corehost_init_t * init = 0x00000208`7b12abc0, int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0)+0x2fc [D:\a\_work\1\s\src\native\corehost\fxr\fx_muxer.cpp @ 146] 
3f 00000072`d677f3b0 00007ffe`351e075f hostfxr!`anonymous namespace'::read_config_and_execute(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * host_command = 0x00000072`d677f6e0 "", struct host_startup_info_t * host_info = <Value unavailable error>, class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * app_candidate = <Value unavailable error>, class std::unordered_map<enum known_options,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > >,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > > > > > * opts = <Value unavailable error>, int new_argc = 0n1, wchar_t ** new_argv = 0x00000208`7b1187f0, host_mode_t mode = apphost (0n2), bool is_sdk_command = false, wchar_t * out_buffer = 0x00000000`00000000 "", int buffer_size = 0n0, int * required_buffer_size = 0x00000000`00000000)+0xa7 [D:\a\_work\1\s\src\native\corehost\fxr\fx_muxer.cpp @ 533] 
40 00000072`d677f4b0 00007ffe`351dea53 hostfxr!fx_muxer_t::handle_exec_host_command(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * host_command = 0x00000072`d677f6e0 "", struct host_startup_info_t * host_info = 0x00000072`d677f700, class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * app_candidate = 0x00000072`d677f620 "D:\程序\ethylene156\Empty-dev\BekuhalnoKawairlunee\BekuhalnoKawairlunee\bin\Debug\net6.0-windows\BekuhalnoKawairlunee.dll", class std::unordered_map<enum known_options,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > >,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> >,std::allocator<std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > > > > > > * opts = 0x00000072`d677f5d0 { size=0x0 }, int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0, int argoff = 0n1, host_mode_t mode = apphost (0n2), bool is_sdk_command = false, wchar_t * result_buffer = 0x00000000`00000000 "", int buffer_size = 0n0, int * required_buffer_size = 0x00000000`00000000)+0x15f [D:\a\_work\1\s\src\native\corehost\fxr\fx_muxer.cpp @ 1018] 
41 00000072`d677f560 00007ffe`351d850b hostfxr!fx_muxer_t::execute(class std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t> > * host_command = 0x00000072`d677f6e0 "", int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0, struct host_startup_info_t * host_info = 0x00000072`d677f700, wchar_t * result_buffer = 0x00000000`00000000 "", int buffer_size = 0n0, int * required_buffer_size = 0x00000000`00000000)+0x483 [D:\a\_work\1\s\src\native\corehost\fxr\fx_muxer.cpp @ 579] 
*** WARNING: Unable to verify checksum for apphost.exe
42 00000072`d677f6a0 00007ff6`a6f61eb8 hostfxr!hostfxr_main_startupinfo(int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0, wchar_t * host_path = 0x00000208`7b127950 "D:\程序\ethylene156\Empty-dev\BekuhalnoKawairlunee\BekuhalnoKawairlunee\bin\Debug\net6.0-windows\BekuhalnoKawairlunee.exe", wchar_t * dotnet_root = 0x00000208`7b127a70 "C:\Program Files\dotnet\", wchar_t * app_path = 0x00000208`7b127780 "D:\程序\ethylene156\Empty-dev\BekuhalnoKawairlunee\BekuhalnoKawairlunee\bin\Debug\net6.0-windows\BekuhalnoKawairlunee.dll")+0xab [D:\a\_work\1\s\src\native\corehost\fxr\hostfxr.cpp @ 61] 
43 00000072`d677f7a0 00007ff6`a6f6222b apphost!exe_start(int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0)+0x8d8 [D:\a\_work\1\s\src\native\corehost\corehost.cpp @ 235] 
44 00000072`d677f970 00007ff6`a6f636d8 apphost!wmain(int argc = 0n1, wchar_t ** argv = 0x00000208`7b1187f0)+0xab [D:\a\_work\1\s\src\native\corehost\corehost.cpp @ 304] 
45 (Inline Function) --------`-------- apphost!invoke_main(void)+0x22 [d:\a01\_work\12\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl @ 90] 
46 00000072`d677f9a0 00007ffe`5eee7614 apphost!__scrt_common_main_seh(void)+0x10c [d:\a01\_work\12\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl @ 288] 
47 00000072`d677f9e0 00007ffe`5fe626a1 KERNEL32!BaseThreadInitThunk+0x14
48 00000072`d677fa10 00000000`00000000 ntdll!RtlUserThreadStart+0x21
```

以上的堆栈十分明确，就是从 PresentationFramework 里触发加载的，核心堆栈如下

```
24 00000072`d677d1d0 00007ffe`3342ebf6 PresentationFramework!System.Windows.Application::ApplicationInit+0x3b
25 00000072`d677d210 00007ffe`34dbae93 PresentationFramework!System.Windows.Application::.cctor+0x66
```

拿到了调用堆栈，依然是需要进行一些业务端的分析，才能了解到具体是哪个模块加载
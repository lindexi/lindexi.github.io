# WPF 异常 NativeWPFDLLLoader.LoadNativeWPFDLL

如果发现出现下面异常，可以尝试重装 .net Framework

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<!-- csdn -->

```csharp
 System.ComponentModel.Win32Exception 异常信息: System.DllNotFoundException 在 MS.Internal.NativeWPFDLLLoader.LoadNativeWPFDLL(UInt16*, UInt16*) 在 MS.Internal.NativeWPFDLLLoader.LoadCommonDLLsAndDwrite() 在 <Module>.CModuleInitialize.{ctor}(CModuleInitialize*, Void ()) 在 <Module>.?A0x721f77f1.CreateCModuleInitialize() 在 <Module>.?A0x721f77f1.??__E?A0x721f77f1@cmiStartupRunner@@YMXXZ() 在 <Module>._initterm_m(Void* ()*, Void* ()*) 在 <Module>.<CrtImplementationDetails>.LanguageSupport.InitializePerAppDomain(<CrtImplementationDetails>.LanguageSupport*) 在 <Module>.<CrtImplementationDetails>.LanguageSupport._Initialize(<CrtImplementationDetails>.LanguageSupport*) 在 <Module>.<CrtImplementationDetails>.LanguageSupport.Initialize(<CrtImplementationDetails>.LanguageSupport*) 异常信息: <CrtImplementationDetails>.ModuleLoadException 在 <Module>.<CrtImplementationDetails>.LanguageSupport.Initialize(<CrtImplementationDetails>.LanguageSupport*) 在 <Module>..cctor() 异常信息: System.TypeInitializationException 在 lindexi.Startup.Main(System.String[]) 
``` 

尝试使用 sfc /scannow 看是否系统坏了，重新安装 .net Framework就好了
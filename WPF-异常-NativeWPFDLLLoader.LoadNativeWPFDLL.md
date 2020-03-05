
# WPF 异常 NativeWPFDLLLoader.LoadNativeWPFDLL

如果发现出现下面异常，可以尝试重装 .net Framework

<!--more-->


<!-- csdn -->

```csharp
 System.ComponentModel.Win32Exception 异常信息: System.DllNotFoundException 在 MS.Internal.NativeWPFDLLLoader.LoadNativeWPFDLL(UInt16*, UInt16*) 在 MS.Internal.NativeWPFDLLLoader.LoadCommonDLLsAndDwrite() 在 <Module>.CModuleInitialize.{ctor}(CModuleInitialize*, Void ()) 在 <Module>.?A0x721f77f1.CreateCModuleInitialize() 在 <Module>.?A0x721f77f1.??__E?A0x721f77f1@cmiStartupRunner@@YMXXZ() 在 <Module>._initterm_m(Void* ()*, Void* ()*) 在 <Module>.<CrtImplementationDetails>.LanguageSupport.InitializePerAppDomain(<CrtImplementationDetails>.LanguageSupport*) 在 <Module>.<CrtImplementationDetails>.LanguageSupport._Initialize(<CrtImplementationDetails>.LanguageSupport*) 在 <Module>.<CrtImplementationDetails>.LanguageSupport.Initialize(<CrtImplementationDetails>.LanguageSupport*) 异常信息: <CrtImplementationDetails>.ModuleLoadException 在 <Module>.<CrtImplementationDetails>.LanguageSupport.Initialize(<CrtImplementationDetails>.LanguageSupport*) 在 <Module>..cctor() 异常信息: System.TypeInitializationException 在 lindexi.Startup.Main(System.String[]) 
``` 

尝试使用 sfc /scannow 看是否系统坏了，重新安装 .net Framework就好了




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
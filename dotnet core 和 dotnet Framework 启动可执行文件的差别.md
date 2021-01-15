# dotnet core 和 dotnet Framework 启动可执行文件的差别

在 Windows 下，使用 .NET Framework 构建出来的应用，可以只有一个可执行文件，在可执行文件里面包含了 IL 代码。而使用 .NET Core 构建出来的应用，将会包含一个 Exe 可执行文件，和对应的 Dll 文件，而 IL 代码将放在 Dll 文件里面。那么使用 .NET Framework 和使用 .NET Core 所输出的 Exe 可执行文件有什么差别

<!--more-->
<!-- CreateTime:2021/1/13 8:54:02 -->

<!-- 发布 -->

在 dotnet core 或 dotnet 5 下，默认输出的 Exe 可执行文件是 AppHost 文件，这是一个纯 Win32 可执行文件，里面不包含 IL 代码。在用户双击运行此 Exe 可执行文件的时候，将会运行起来这个 Win32 应用，在这里面将调用起 CLR 引擎，执行放在 Dll 的 IL 代码。此部分逻辑相对复杂，详细请看 [dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

而在 .NET Framework 里面，根据 [Managed Execution Process 官方文档](https://docs.microsoft.com/en-us/dotnet/standard/managed-execution-process?WT.mc_id=DX-MVP-5003606 ) 可以了解到，输出的 Exe 可执行文件里面，格式是符合 PE 格式的文件，但是在 PE 文件中包含了从元数据和 MSIL 代码。在双击可执行程序运行的时候，首先进入的 operating system loader 将会判断 PE 文件的 COFF 头内容，通过 COFF 头识别这个可执行文件是否 .NET Framework 可执行文件，也就是说 .NET Framework 生成的可执行文件是由系统进行特别支持的。此时将会加载 mscoree.dll 进行执行，通过 `_CorValidateImage` 和 `_CorImageUnloading` 分别用来通知 operating system loader 托管模块的映像的加载和卸载。其中在 `_CorValidateImage` 中将执行确保该代码是有效的托管代码以及将映像中的入口点更改为运行时中的入口点。而在 x64 中，还会在 `_CorValidateImage` 中通过在内存中修改映像的 PE32 为 PE32+ 格式。也因为 .NET Framework 应用是依靠系统的特殊处理，因此 .NET Framework 又有一个原因耦合了系统环境，这和 .NET Core 的启动有着本质的差别

文件内容的差别是：

- .NET Core: 纯 Win32 的 PE 格式文件，不包含 IL 逻辑。包含 IL 逻辑的放在额外的 Dll 文件
- .NET Framework: 稍微特殊的 Win32 的 PE 格式文件，包含了特殊 COFF 头内容用来标识这是 .NET Framework 文件。在 PE 格式文件里面包含了 IL 逻辑

启动的时候的差别是：

- .NET Core: 作为传统的 Win32 应用启动，在启动过程中加载 CLR 引擎，然后通过 CLR 引擎执行 IL 逻辑
- .NET Framework: 由系统根据 COFF 头判断这是 .NET Framework 应用，通过特殊手段启动，使用系统的 mscoree.dll 进行初始化

这就是 .NET Framework 和 .NET Core 启动的可执行文件的差别，以及执行的差别

现在的 .NET Framework 的运行时大部分逻辑都没有开源（我即使能通过MVP权限拿到我也不敢在这里吹）因此只能通过官方公开的文档了解到细节，而 .NET Core 是完全开源的，因此我对 .NET Core 里面的逻辑相对来说更了解。好在后续将会统一使用为完全开源的 .NET 5 以及后续版本，所以即使对 .NET Framework 的执行细节不了解，问题也许不大

关于 .NET Core 底层接地气的书籍，我推荐农夫的 《.NET Core底层入门》 这本书。而关于内存相关，我推荐伟民哥翻译的 .NET内存管理宝典 - 提高代码质量、性能和可扩展性 这本书

参考

[dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

[dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87%E8%87%AA%E5%B7%B1%E5%86%99%E4%B8%80%E4%B8%AA-dotnet-host-%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

[Managed Execution Process](https://docs.microsoft.com/en-us/dotnet/standard/managed-execution-process?WT.mc_id=DX-MVP-5003606 ) 
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

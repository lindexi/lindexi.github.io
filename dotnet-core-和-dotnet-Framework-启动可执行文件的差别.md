
# dotnet core 和 dotnet Framework 启动可执行文件的差别

在 Windows 下，使用 .NET Framework 构建出来的应用，可以只有一个可执行文件，在可执行文件里面包含了 IL 代码。使用 .NET Core 构建出来的应用，将会包含一个 Exe 可执行文件，和对应的 Dll 文件，而 IL 代码将放在 Dll 文件里面。那么使用 .NET Framework 和使用 .NET Core 所输出的 Exe 可执行文件有什么差别，本文将从文件格式以及启动过程两个方面给大家聊聊这两个的不同

<!--more-->



<!-- 发布 -->

与封闭的 dotnet framework 不相同的是，咱可以从 GitHub 上获取完全的 dotnet core 整个的源代码。也因此咱能了解到的 dotnet core 的细节将会比 dotnet framework 多得多。在本文开始之前，必须要说明的是，本文非入门向，而且本文的内容是需要在很多的限制条件下才成立。在经过了这么多年的发展，无论是 dotnet framework 还是 dotnet core 都有着各自的不同的多个版本甚至的多个分支，每个之间都有着很多差别。而限于我的技术和考古能力，我仅仅能聊的只有最通用的部分

当前有很多书籍在讲 dotnet core 的底层，我在这里将这些书籍推荐给期望了解更多底层细节的伙伴：农夫的 《.NET Core底层入门》 以及伟民哥翻译的 《.NET内存管理宝典 - 提高代码质量、性能和可扩展性》 这两本书。本文的很多细节出处都来自于这两本书

本文所指的 dotnet core 包括了 dotnet core 以及 dotnet 5 等多个版本，不讨论加入 Mono 以及加入 .NET Native 和单文件发布等科技。本文的 dotnet framework 指的是 dotnet framework 4.0 到 4.8 的版本，其他版本不在本文范围内，根据我的考古，更古老的 dotnet framework 有不同的行为，但我缺乏足够的依据，因此也不在放在本文

先从文件的格式开始聊起。在咱写一个默认的 .NET Framework 控制台应用的时候，在 VisualStudio 上进行 Debug 调试构建输出，此时将可以从输出文件路径上看到仅仅只有一个 EXE 可执行文件，而没有 DLL 动态链接库文件。这个输出的 Exe 可执行文件是一个符合标准的 PE 格式的文件。而 PE （Portable Executable）格式文件是微软 Win32 环境可执行文件的标准文件格式。也就是说使用 .NET Framework 输出的可执行文件和其他 Win32 可执行文件的文件格式是相同的。尽管格式上是相同的，微软在 Windows 下依然对 .NET Framework 应用做了特别的处理，因为在 .NET Framework 输出的可执行文件里面，包含了从元数据和 MSIL 代码，换句话说就是真正的逻辑是包含在 MSIL 代码里面，而不是作为本机代码的存在。这就需要 Windows 系统在用户执行 .NET Framework 可执行文件的时候进行一些特殊的处理

那既然 .NET Framework 的可执行文件在执行时需要 Windows 做特殊的处理，那么 Windows 如何了解到这是一个需要处理的 .NET Framework 应用？根据 [Managed Execution Process 官方文档](https://docs.microsoft.com/en-us/dotnet/standard/managed-execution-process?WT.mc_id=DX-MVP-5003606 ) 可以了解到，在 .NET Framework 的输出可执行文件里面，在 PE 文件的 COFF 头内容添加了特殊的内容，用来标识这是一个 .NET Framework 应用

在运行 .NET Framework 的可执行文件的时候，首先进入的 operating system loader 将会判断 PE 文件的 COFF 头内容，通过 COFF 头识别这个可执行文件是否 .NET Framework 可执行文件。对于 .NET Framework 可执行文件而言，将会加载 mscoree.dll 进行执行，通过 `_CorValidateImage` 和 `_CorImageUnloading` 分别用来通知 operating system loader 托管模块的映像的加载和卸载。其中在 `_CorValidateImage` 中将执行确保该代码是有效的托管代码以及将映像中的入口点更改为运行时中的入口点。而在 x64 中，还会在 `_CorValidateImage` 中通过在内存中修改映像的 PE32 为 PE32+ 格式。也因为 .NET Framework 应用是依靠系统的特殊处理，因此 .NET Framework 又有一个原因耦合了系统环境，这和 .NET Core 的启动有着本质的差别

回到 .NET Core 下，依然是通过 VisualStudio 在 Debug 下构建输出一个控制台的应用。此时可以看到构建输出将会包含一个 Exe 和对应的 Dll 文件。通过 [dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html ) 可以了解到，在 .NET Core 的默认输出的 Exe 可执行文件是 AppHost 文件，这是一个纯 Win32 可执行文件，里面不包含 IL 代码。而业务逻辑的 IL 代码是存放在 DLL 里面

在 .NET Core 下，输出的 Exe 可执行文件其实是通过预先已经构建完毕的模板文件，进行二进制修改替换生成的文件。在构建的过程中，将会从 `C:\Program Files\dotnet\sdk\5.0.100\AppHostTemplate\` 文件夹里面将 apphost.exe 文件拷贝到项目的 obj 文件夹下。然后执行 HostWriter 构建过程命令，将 AppHost 中的特定二进制内容替换为具体项目的信息，接着拷贝到最终输出文件路径。当然，上文的 AppHostTemplate 文件夹路径会根据你所安装的 dotnet sdk 版本而变化。那么这个输出的 AppHost 文件是谁构建的，作用又是什么？细节部分如下

通过开源的 .NET Core 仓库，可以了解到在 `dotnet runtime\src\installer\corehost\` 文件夹里面，其实就是 AppHost 文件的核心逻辑。通过阅读 `dotnet runtime\src\installer\corehost\corehost.cpp` 文件可以了解到在 AppHost 文件在执行过程中所执行的过程。在 `dotnet runtime\src\installer\corehost\corehost.cpp` 文件的 `exe_start` 大概就是整个可执行文件的入口方法了，在这里实现的功能将包含使用 hostfxr 和 hostpolicy 来托管执行整个 dotnet 进程，以及主函数的调起。而在使用托管之前，需要先寻找 dotnet_root 也就是 dotnet 框架用来承载整个 dotnet 进程。而不同的项目之间有着不同的项目信息，需要在执行过程中进行动态配置。这就是在构建过程中 HostWriter 构建过程所执行的逻辑，将预先构建完成的 AppHost 中的部分内容替换为具体项目的信息，同时给 AppHost 嵌入 Win32 清单，如图标等内容

在 .NET Core 中，在 SDK 里面已经包含了将 `dotnet runtime\src\installer\corehost\` 文件夹里面的 AppHost 构建输出的 AppHost.exe 文件。在此文件里面由以下代码定义了部分模板替换内容

```c++
#define EMBED_HASH_HI_PART_UTF8 "c3ab8ff13720e8ad9047dd39466b3c89" // SHA-256 of "foobar" in UTF-8
#define EMBED_HASH_LO_PART_UTF8 "74e592c2fa383d4a3960714caef0c4f2" // 这两句代码就是 foobar 的 UTF-8 二进制的 SHA-256 字符串
#define EMBED_HASH_FULL_UTF8    (EMBED_HASH_HI_PART_UTF8 EMBED_HASH_LO_PART_UTF8) // NUL terminated

bool is_exe_enabled_for_execution(pal::string_t* app_dll)
{
    constexpr int EMBED_SZ = sizeof(EMBED_HASH_FULL_UTF8) / sizeof(EMBED_HASH_FULL_UTF8[0]);
    // 这里给的是就是最长 1024 个 byte 的 dll 名，加上一个 \0 一共是 1025 个字符
    constexpr int EMBED_MAX = (EMBED_SZ > 1025 ? EMBED_SZ : 1025); // 1024 DLL name length, 1 NUL

    // 这就是定义在 AppHost.exe 二进制文件里面的一段空间了，长度就是 EMBED_MAX 长度，内容就是 c3ab8ff13720e8ad9047dd39466b3c8974e592c2fa383d4a3960714caef0c4f2 这段字符串
    static char embed[EMBED_MAX] = EMBED_HASH_FULL_UTF8;     // series of NULs followed by embed hash string

    static const char hi_part[] = EMBED_HASH_HI_PART_UTF8;
    static const char lo_part[] = EMBED_HASH_LO_PART_UTF8;

    // 将 embed 的内容复制到 app_dll 变量里面
    pal::clr_palstring(embed, app_dll);
}


int exe_start(const int argc, const pal::char_t* argv[])
{
	// 读取嵌入到二进制文件的 App 名，也就是 dotnet 的入口 dll 路径，可以是相对也可以是绝对路径
    pal::string_t embedded_app_name;
    if (!is_exe_enabled_for_execution(&embedded_app_name))
    {
        trace::error(_X("A fatal error was encountered. This executable was not bound to load a managed DLL."));
        return StatusCode::AppHostExeNotBoundFailure;
    }

    // 将 embedded_app_name 的内容赋值给 app_path 变量，这个变量的定义代码我没有写
    append_path(&app_path, embedded_app_name.c_str());

    const pal::char_t* app_path_cstr = app_path.empty() ? nullptr : app_path.c_str();
    // 跑起来 dotnet 应用
    rc = hostfxr_main_bundle_startupinfo(argc, argv, host_path_cstr, dotnet_root_cstr, app_path_cstr, bundle_header_offset);
}
```

也如上面代码所述，在 AppHost 文件里面将会通过 hostfxr_main_bundle_startupinfo 跑起来 dotnet 应用，加载 CLR 执行引擎，执行 DLL 里面的 IL 逻辑

在具体项目构建过程中，将通过 HostWriter 构建过程，执行如下逻辑，将如上面代码的 foobar 的 UTF-8 二进制的 SHA-256 字符串替换为具体项目的路径

```csharp
    /// <summary>
    /// Embeds the App Name into the AppHost.exe
    /// If an apphost is a single-file bundle, updates the location of the bundle headers.
    /// </summary>
    public static class HostWriter
    {
        /// <summary>
        /// 这就是 AppHost 的 foobar 的 UTF-8 二进制的 SHA-256 字符串
        /// </summary>
        private const string AppBinaryPathPlaceholder = "c3ab8ff13720e8ad9047dd39466b3c8974e592c2fa383d4a3960714caef0c4f2";
        private static readonly byte[] AppBinaryPathPlaceholderSearchValue = Encoding.UTF8.GetBytes(AppBinaryPathPlaceholder);

        /// <summary>
        /// Create an AppHost with embedded configuration of app binary location
        /// </summary>
        /// <param name="appHostSourceFilePath">The path of Apphost template, which has the place holder</param>
        /// <param name="appHostDestinationFilePath">The destination path for desired location to place, including the file name</param>
        /// <param name="appBinaryFilePath">Full path to app binary or relative path to the result apphost file</param>
        /// <param name="windowsGraphicalUserInterface">Specify whether to set the subsystem to GUI. Only valid for PE apphosts.</param>
        /// <param name="assemblyToCopyResorcesFrom">Path to the intermediate assembly, used for copying resources to PE apphosts.</param>
        public static void CreateAppHost(
            string appHostSourceFilePath,
            string appHostDestinationFilePath,
            string appBinaryFilePath,
            bool windowsGraphicalUserInterface = false,
            string assemblyToCopyResorcesFrom = null)
        {
            var bytesToWrite = Encoding.UTF8.GetBytes(appBinaryFilePath);
            if (bytesToWrite.Length > 1024)
            {
                throw new AppNameTooLongException(appBinaryFilePath);
            }

            void RewriteAppHost()
            {
                // Re-write the destination apphost with the proper contents.
                using (var memoryMappedFile = MemoryMappedFile.CreateFromFile(appHostDestinationFilePath))
                {
                    using (MemoryMappedViewAccessor accessor = memoryMappedFile.CreateViewAccessor())
                    {
                        BinaryUtils.SearchAndReplace(accessor, AppBinaryPathPlaceholderSearchValue, bytesToWrite);

                        appHostIsPEImage = PEUtils.IsPEImage(accessor);

                        if (windowsGraphicalUserInterface)
                        {
                            if (!appHostIsPEImage)
                            {
                                throw new AppHostNotPEFileException();
                            }

                            PEUtils.SetWindowsGraphicalUserInterfaceBit(accessor);
                        }
                    }
                }
            }

            // 忽略代码
        }
    }
```

那为什么在 dotnet 里面选择的是预先构建出来 AppHost 文件，将 AppHost 文件放在 SDK 里面。在构建代码的时候通过构建过程将 AppHost 文件的部分二进制替换而输出为最终可执行文件？原因就是 dotnet core 期望能做到让构建过程尽可能简单，同时又期望能支持更多的平台。通过预先构建出来可执行二进制文件，可以在构建出来的二进制文件尽可能使用 Native 的逻辑，而一旦使用了 Native 的逻辑就意味着构建环境有足够的要求。但是在开发者端，其实很没有必要去安装那么多的构建环境，因此就预先构建出来二进制文件，这样开发者端就能专注于构建 dotnet 的逻辑，而不需要为了构建入口的可执行文件而安装大量的构建环境

在 dotnet core 里面，所有的 IL 逻辑存放在独立的 DLL 里面，在 Windows 下的可执行文件仅仅是 AppHost 文件，是一个没有被系统特殊处理的 Win32 可执行文件。在运行过程中，将在一开始执行 AppHost 的本机逻辑，在 AppHost 的 Native 逻辑里面将跑起来 dotnet 引擎，加载 DLL 里面的 IL 逻辑，然后将舞台交给咱的业务逻辑

回顾一下 dotnet core 和 dotnet framework 的可执行文件的差别

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
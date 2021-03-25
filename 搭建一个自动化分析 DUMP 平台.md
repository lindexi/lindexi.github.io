# 搭建一个自动化分析 DUMP 平台

本文来告诉大家如何一步步搭建一个 DUMP 分析平台，核心是用来分析桌面端的应用软件，如 WPF 软件的 DUMP 文件。在开始之前需要说明的是，如果桌面端软件使用纯 WPF 实现，中途没有调用不安全的 C++ 库，那么 DUMP 平台几乎无用，原因是 WPF 是 .NET 应用，而 .NET 是安全的，除非是系统环境问题，否则依靠捕获异常所拿到的信息就完全超过了 DUMP 能获取的信息。因此本文的核心功能是提供给调用了不安全的 C++ 等语言编写的库的桌面端软件 DUMP 分析平台

<!--more-->
<!-- CreateTime:2021/3/23 18:56:30 -->

<!-- 发布 -->

对于 C++ 等不安全语言编写的逻辑，将会比较多依赖 DUMP 的调试。但对于 dotnet 应用来说，依靠异常就完全足够了，只要遵循规范，那么基本只有内存爆了、无限递归等很有限的几个问题才能玩炸，其他情况都能稳稳接住

在搭建 DUMP 平台的工作中，可以分为两个部分，第一个部分需要做到自己的构建平台上，在构建的时候需要保存足够的符号文件用于后续调试。另一个是通过 WinDbg 进行的稍微自动化的自动读取 DUMP 信息导出的工作，以及通过这些导出的内容再次做处理的工作

敲黑板，如果是 .NET 的应用，如 WPF 或 WinForms 应用，还请优先搭建日志模块，将异常记录到日志将能解决几乎所有的问题。如果靠异常也不能定位，那么靠 DUMP 基本也是不能的，除非你更熟悉 DUMP 而不是 .NET 机制。更多关于 .NET 的异常处理请看 [一文看懂 .NET 的异常处理机制、原则以及最佳实践 - walterlv](https://blog.walterlv.com/post/dotnet-exception.html )

## 构建平台保存符号

在使用 DUMP 调试的过程中，很重要一定就是需要有符号 PDB 文件。在后续的分析过程需要用到符号文件。在自己的构建平台，如 GitHub 的 Runner 自托管服务上需要自行部署

只需要将保存的符号文件和 dll 和 exe 等文件，开启文件服务器或者作为网络磁盘挂载等都可以作为符号服务器。关于开启文件服务器，我推荐使用此方法 [dotnet serve 一句话开启文件服务器 通过 HTTP 将文件共享给其他设备](https://blog.lindexi.com/post/dotnet-serve-%E4%B8%80%E5%8F%A5%E8%AF%9D%E5%BC%80%E5%90%AF%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8-%E9%80%9A%E8%BF%87-HTTP-%E5%B0%86%E6%96%87%E4%BB%B6%E5%85%B1%E4%BA%AB%E7%BB%99%E5%85%B6%E4%BB%96%E8%AE%BE%E5%A4%87.html)

想要被 WinDbg 所使用的符号文件服务器，需要将符号文件按照一定的格式存放在文件夹中，格式如下

```
xx.dll -> GUID -> xx.dll
```

此时需要用到 symstore 工具，这个工具和 WinDbg 工具都是放在 WDK 工具集里面，建议不要使用绿色版，而是自行安装 WDK 工具集。以下是安装 WDK 工具集的方法

先进入到官方文档 [Download the Windows Driver Kit (WDK) - Windows drivers](https://docs.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk?WT.mc_id=WD-MVP-5003260  ) 按照步骤，先安装 VS 然后安装 Windows SDK 然后安装 WDK 工具集

安装 WDK 推荐安装到默认的路径，也就是 `c:\Program Files (x86)\Windows Kits\10` 文件夹

接着可以在打包平台上使用 symstore 工具将自己的应用存放到符号服务器的文件夹

```
symstore.exe add /r /t [项目名] /s [符号服务器文件夹] /f [本地构建输出文件夹] 
```

如符号文件夹是 `C:\lindexi\Symbol` 文件夹，而本地构建输出文件夹是 `F:\code\lindexi\lindexi\bin\Debug\net5.0\` 文件夹，此项目名是 LindexiDoubi 那么命令如下

```
symstore.exe add /r /t LindexiDoubi /s "C:\lindexi\Symbol" /f "F:\code\lindexi\lindexi\bin\Debug\net5.0" 
```

上面命令的 `add` 就是添加保存的文件，而 `/r` 表示 `/f` 的本地构建输出文件夹的内容需要递归文件夹，也就是获取文件夹里面的文件夹的内容

调用上面命令之后，将会在 `C:\lindexi\Symbol` 文件夹里面创建如下内容

```
│  pingme.txt
│  
├─000Admin
│      0000000001
│      history.txt
│      lastid.txt
│      server.txt
│      
├─lindexi.exe
│  └─F78EFB318000
│          lindexi.exe
│          refs.ptr
│          
└─lindexi.pdb
    └─4E7E5EA6C9414D2B8B0204E7D146D3901
            lindexi.pdb
            refs.ptr
```

在新版本的 .NET 里面，将使用 Portable PDB 这将会让旧版本的 symstore 失败。也就是说如果你的 symstore 无法存储 PDB 文件时，请确定你的 symstore 是使用最新的 WDK 工具

此时只需要在 000Admin 文件夹所在的文件夹，如 `C:\lindexi\Symbol` 文件夹开启文件服务器，那么此文件夹服务器就是符号服务器

## 使用 WinDbg 分析

在从用户端或开发端收集到 DUMP 文件之后，可以利用上面步骤创建出来的符号服务器和 DUMP 文件借助 WinDbg 来分析

我推荐你在自动分析服务器上，先使用 WinDbg 手动分析一个 DUMP 用来确定你的本地环境，以及让本地缓存足够的符号文件。我的符号文件大概有 10G 左右，大部分都是各个版本系统的文件

在 Windows 下可以说 WinDbg 是最强的调试工具，自然 WinDbg 工具也可以了命令行版本的自动化方法，可以将命令通过命令行方式传入到 WinDbg 中，让 WinDbg 执行，然后输出为本地文件。这样做的优势在于可以利用 WinDbg 加上预定义的命令进行自动化调试 DUMP 文件，然后输出的日志可以进入下一个步骤的处理，获取到信息

每个团队的需求都不同，因此也不存在万能的预定义命令。如我所在的团队，只需要处理甩锅就可以了，我只需要了解到当前 DUMP 的大概原因，通过分类算法处理 WinDbg 输出的文件，然后分为不同的其他团队就可以了

在 WinDbg 中，可以使用 -c 命令，让 WinDbg 执行预定义的命令。可以使用 `-z` 告诉 WinDbg 将要调试的 DMP 文件路径。通过 `-y` 命令可以指定上面步骤创建的符号服务器。通过 `-logo` 命令可以让 WinDbg 输出日志文件

格式大概如下

```
"C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\windbg.exe" -z [DUMP文件路径] -y "SRV*[本地缓存自定义符号文件夹]*[自己搭建的符号服务器]" -c "$<[预定义命令文件]" -logo [输出的日志文件]
```

例如 WinDbg 是通过 WDK 安装到默认的文件夹，可以使用 `C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\windbg.exe` 调试工具，当然也有 x86 的版本

需要调试的 DMP 文件是 `F:\temp\foo.dmp` 文件，而自己搭建的符号服务器以及需要做的本地符号缓存文件夹分别是 `http://localhost:5000` 和 `F:\lindexi\AppSymCachePath` 文件夹。预定义的 WinDbg 命令放在 `C:\lindexi\WinDbgFile.txt` 文件。输出的日志放在 `C:\lindexi\log.txt` 文件

那么命令如下


```
"c:\Program Files (x86)\Windows Kits\10\Debuggers\x64\windbg.exe" -z F:\temp\foo.dmp -y "SRV*F:\lindexi\AppSymCachePath*http://localhost:5000" -c "$<C:\lindexi\WinDbgFile.txt" -logo C:\lindexi\log.txt
```

如果在 -c 命令中，传入的是命令本身，如 `-c !analyze;!clrstack;qq;` 而在 WinDbg 提示 `The command line argument cannot specify more than one kind of debugging to start` 那么就是少加了引号了。对于上面命令的路径都需要加上引号

```
-c "!analyze -v;!clrstack;qq;"
```

在 `-c` 命令中，可以加上的参数是命令，或者存放命令的文件。存放命令的文件，将一条命令放在一行里面，如下面代码

```
!analyze; -v
!clrstack
q
```

大家需要根据自己的需求，修改自己的命令文件

通过上面方法就可以自己搭建 DUMP 平台，自己需要做的就是先自己本地先跑一下，包括自己创建符号服务器，自己命令行 WinDbg 调试一下。然后就是开发自动化的逻辑

因为 WinDbg 是一个十分强大的调试工具，需要的知识也特别多，所以还请大家准备足够的精力再来开这个坑。如果不需要批量调试 DUMP 文件，不如试试将 DUMP 拖入到 VisualStudio 中，使用熟悉的 VS 调试

和大家推荐 DUMP 调试群 169225649

[a (Run Script File) - Windows drivers](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-----------------------a---run-script-file-?WT.mc_id=WD-MVP-5003260 )

[Debugging Tools for Windows (WinDbg, KD, CDB, NTSD) - Windows drivers](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger?WT.mc_id=WD-MVP-5003260 )

[SymStore Command-Line Options - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/symstore-command-line-options?WT.mc_id=WD-MVP-5003260 )

[Using SymStore - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/using-symstore?WT.mc_id=WD-MVP-5003260 )

[.NET Framework system requirements](https://docs.microsoft.com/en-us/dotnet/framework/get-started/system-requirements?WT.mc_id=WD-MVP-5003260 )

[Download the Windows Driver Kit (WDK) - Windows drivers](https://docs.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk?WT.mc_id=WD-MVP-5003260  )

[搭建Windows符号服务器](https://xyz1001.xyz/articles/22247.html )

[windbg 边学边记attach 进程和open dump的两个方式查看线程的占用cpu资源](https://www.cnblogs.com/zuochanzi/p/6912808.html)

[core/portable_pdb.md at main · dotnet/core](https://github.com/dotnet/core/blob/main/Documentation/diagnostics/portable_pdb.md )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

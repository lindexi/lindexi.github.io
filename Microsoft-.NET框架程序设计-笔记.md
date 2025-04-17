
# Microsoft .NET框架程序设计 笔记



托管模块的组成，如何设置就是识别 DLL 如何设置就是 窗口
使用 IL 在 ngen 本机代码，尝试让软件使用 ngen 做发布查看效率
<!-- 草稿 -->
这个需要说到了使用 csc 的两个不同编译命令，在 csc 可以编译出控制台用户界面，和图形界面 GUI 两个不同的程序。通过在编译过程指定对应的开关可以编译出不同的软件。
如果使用下面的代码`/t:exe`的开关就会编译出控制台软件，而指定`/t:winexe`可以使C#编译器创建一个图形界面软件。
在 csc 创建的软件的文件，就是平时看到的 exe 是一个 PE 文件，对于 PE 文件意味着在 x86 和 x64 的设备上可以加载使用这个文件。当然如果大家指定了特定的平台，如指定了 x64 就不能在 x86 的设备正常使用，但是这个文件还是可以被系统加载使用。
一个 PE 文件是包括
如果 DLL 多了元数据占用比例太大，如 lindexi.exe 的元数据占用了文件大部分，而 IL 代码只占用 20 字节。所以将功能分的太小放在多个 dll 里会带来更多的元数据信息，减少 IL 代码比例，从而降低了性能。
只有在执行到对应的方法的时候才会去读取方法所在的 dll ，如果大家的软件在启动过程需要用到如 MEF这样的框架，就会在启动的过程扫整个需要加载的 dll 这就让所有的 dll 在启动的过程都需要加载。如果这时用户的设备使用的不是固态硬盘，那么读取 dll 这个文件读取过程将会非常耗时，在机械硬盘，如果是读取小文件，读取速度将会非常慢。
des 加密需要 DesCbc 内容是 8 的倍数，不然出现
```csharp
System.Exception:“提供给请求操作的用户缓冲区无效。 (Exception from HRESULT: 0x800706F8)”
```




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
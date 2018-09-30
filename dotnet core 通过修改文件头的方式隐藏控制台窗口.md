# dotnet core 通过修改文件头的方式隐藏控制台窗口

在带界面的 dotnet core 程序运行的时候就会出现一个控制台窗口，本文告诉大家使用最简单方法去隐藏控制台窗口。

<!--more-->
<!-- 标签：Avalonia,Roslyn，dotnetcore -->

最近在使用 Avalonia 开发，这是一个支持 dotnet framework 和 dotnet core 的框架，在开发的过程发现启动的窗口居然会出现控制台窗口。

在 VisualStudio 2017 的格式，如果是 dotnet framework 的程序，在没有设置 `OutputType` 为 `WinExe` 的是时候，启动程序会显示一个控制台窗口。

虽然在 dotnet framework 程序可以简单通过设置`OutputType` 为 `WinExe` 解决。如果这时使用跨平台多项目，就会看到 dotnet core 项目依旧会显示黑色窗口

最简单的方法是通过修改 PE 文件的方式，在 [NSubsys](https://github.com/jmacato/NSubsys/blob/master/NSubsys.csproj )就是编写了一个 Task 用来在编译完成获取 Exe 修改 PE 文件格式，隐藏黑色窗口

安装 [NSubsys](https://github.com/jmacato/NSubsys/blob/master/NSubsys.csproj ) 的方法就是从 Nuget 搜索 [NSubsys](https://github.com/jmacato/NSubsys/blob/master/NSubsys.csproj ) 然后使用 `dotnet publish `一下就可以了

<!-- ![](image/dotnet core 通过修改文件头的方式隐藏控制台窗口/dotnet core 通过修改文件头的方式隐藏控制台窗口0.png) -->

![](http://image.acmx.xyz/lindexi%2F20187242045550)

如果想知道为什么你安装了一个 Nuget 就可以帮你修改请看[如何创建一个基于 MSBuild Task 的跨平台的 NuGet 工具包 - walterlv](https://walterlv.github.io/post/create-a-cross-platform-msbuild-task-based-nuget-tool.html )

虽然知道了使用这个方法可以隐藏控制台，但是这里还是需要告诉大家一些原理。

在使用 C# 编译器，可以通过 csc 加上一个开关 `/t:` 告诉 csc 当前编译出来的是 控制台界面还是图形界面。通过下面不同的代码可以创建不同的软件

```csharp
/t:exe 创建控制台软件
/t:winexe 创建图形界面软件
```
创建的软件的文件如 exe 或 dll 都是 PE 文件，在 PE 文件有一个 Subsystem 的字段表示了这个 PE 文件是控制台软件还是图形界面。在 PE 文件通过 `Subsystem` 可以告诉系统，现在打开的软件是控制台软件还是图形界面软件，这个值有很多个表示，其中可以使用`IMAGE_SUBSYSTEM_WINDOWS_GUI`表示这是一个 GUI 软件，通过`IMAGE_SUBSYSTEM_WINDOWS_CUI`可以表示这是一个控制台软件。

所以只需要通过修改 PE 文件的方式去告诉系统，这个软件是图形软件，就不会显示控制台。

更多关于 PE 文件请看 [windows PE文件结构及其加载机制 - CSDN博客](https://blog.csdn.net/liuyez123/article/details/51281905 )

[PE文件结构详解 - CSDN博客](https://blog.csdn.net/huanjieshuijing/article/details/5874365 )

[Windows下Console和Win32程序差异-HelloWorld-51CTO博客](http://blog.51cto.com/vanshell/422909 )

[深入理解 Win32 PE 文件格式 - 国立秀才 - 博客园](https://www.cnblogs.com/guolixiucai/p/4809820.html )

更多阅读

[你应该知道的程序集版本 - WeihanLi - 博客园](https://www.cnblogs.com/weihanli/p/assembly-version.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

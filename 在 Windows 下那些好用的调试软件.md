# 在 Windows 下那些好用的调试软件

在开发 Windows 程序经常需要调试软件，本文介绍 Windows Sysinternals 的好用的工具。

<!--more-->
<!-- CreateTime:2018/9/20 17:37:01 -->

<!-- csdn -->
<div id="toc"></div>
<!-- 标签：调试 -->


## Procmon Monitor

可以监听程序对所有文件、网络、注册表的访问，程序创建的线程。

可以用来调试软件找不到 dll 的文件，可以调试软件在启动过程访问的文件。

可以调试软件访问哪些注册表。

更多介绍请看[Procmon Monitor简介](https://blog.csdn.net/zhongguoren666/article/details/7087749 )，如何使用参见[Sysinternals系列工具之Process Monitor用法](https://blog.csdn.net/mvtechnology/article/details/6971786 )

[Process Monitor中文手册 - CSDN博客](https://blog.csdn.net/whatday/article/details/8758380 )

下载：[Process Monitor - Windows Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon )

## ListDLLs

用来列出程序加载的全部的 dll ，这是一个控制台应用，需要在 cmd 下使用。

找到所有加载 dll 的方法是在直接运行

```csharp
Listdlls.exe
```

找到指定的进程使用的dll可以传入进程名或进程id来找到

```csharp
Listdlls.exe [processname|pid]
```

如找到 tim 的加载 dll 是哪些

```csharp
Listdlls.exe tim.exe
```

反过来找到某个 dll 被哪些进程运行

```csharp
listdlls -d dllname
```

如找到 ntdll.dll 被哪些进程打开

```csharp
listdlls.exe -d ntdll.dll
```

更多工具请看 [微软极品工具箱-Sysinternals Suite](https://www.cnblogs.com/zhaoqingqing/p/5641934.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

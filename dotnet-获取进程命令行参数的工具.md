
# dotnet 获取进程命令行参数的工具

在 Windows 下，想要获取指定进程或所有进程的命令行参数，此时需要一些工具的辅助。本文安利大家一个好用的 dotnet 工具，用于获取 Win32 进程的命令行参数

<!--more-->


<!-- CreateTime:2020/8/4 19:39:19 -->



这是一个 dotnet 工具，因此安装特别方便，只需要在命令行输入下面代码就可以

```
dotnet tool install -g dotnetCampus.Win32ProcessCommandViewer.Tool
```

安装完成，可以使用下面代码使用这个工具

```
pscv
```

这个命令不添加任何参数将输出本机所有进程，和进程的命令行参数，有些有趣的进程拿不到就不输出

输出指定进程名的进程的命令行:

```csharp
pscv -n [Process Name]
```

输出指定进程 Id 的进程的命令行:

```csharp
pscv -i [Process Id]
```

这个工具完全开源，请看 [https://github.com/dotnet-campus/dotnetCampus.Win32ProcessCommandViewer](https://github.com/dotnet-campus/dotnetCampus.Win32ProcessCommandViewer)

用到的技术请看

[dotnet 获取指定进程的输入命令行](https://blog.lindexi.com/post/dotnet-%E8%8E%B7%E5%8F%96%E6%8C%87%E5%AE%9A%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BE%93%E5%85%A5%E5%91%BD%E4%BB%A4%E8%A1%8C.html)

[dotnet 通过 WMI 获取指定进程的输入命令行](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E6%8C%87%E5%AE%9A%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BE%93%E5%85%A5%E5%91%BD%E4%BB%A4%E8%A1%8C.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
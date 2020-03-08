# 如何让软件以 System 权限运行

本文只是告诉大家调试 System 运行软件时可以如何快速让一个应用以 System 权限运行。使用 PsExec 可以让软件以 System 账户运行

<!--more-->
<!-- CreateTime:2020/2/8 19:22:24 -->

<!-- 发布 -->

我最近开发的 WPF 小工具需要在用户端被一个 System 权限的服务运行，我想调试这个 System 权限运行的程序，此时快速的方法是通过 PsExec 运行程序，同时在程序里面输出文件日志

从官网 https://docs.microsoft.com/en-us/sysinternals/downloads/psexec 下载最新版的 PsExec 工具，通过管理员权限运行

假设需要运行的程序是 foo.exe 那么通过下面命令行可以让 foo.exe 以 System 权限运行

```
psexec -s foo.exe
```

那么用 system 帐号运行的程序使用下面代码返回的值有什么不同

```csharp
System.Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData)
```

使用 SYSTEM 账户将映射 C:\windows\system32\config\systemprofile\appdata 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

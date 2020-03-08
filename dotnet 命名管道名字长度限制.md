# dotnet 命名管道名字长度限制

在 dotnet 里面可以使用 NamedPipeClientStream 作为命名管道，此时的命名有长度限制，要求在 256 字符之内

<!--more-->
<!-- CreateTime:2019/9/2 11:54:50 -->

<!-- csdn -->

从官方[文档](https://docs.microsoft.com/en-us/windows/win32/ipc/pipe-names) 可以看到限制 256 字符内

```csharp
The entire pipe name string can be up to 256 characters long
```

详细请看

[NamedPipeClientStream Constructor (System.IO.Pipes)](https://docs.microsoft.com/en-us/dotnet/api/system.io.pipes.namedpipeclientstream.-ctor?wt.mc_id=MVP )

[Pipe Names - Windows applications](https://docs.microsoft.com/en-us/windows/win32/ipc/pipe-names?wt.mc_id=MVP )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

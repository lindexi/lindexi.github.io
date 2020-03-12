# dotnet 入门到放弃 使用 .NET Core 卸载工具

我从 dotnet core 1 的版本到 3.1 的版本，中间安装了超级多的预览版，此时我的硬盘已经空间不够了。干的漂亮的 dotnet 提供了 .NET Core 卸载工具专门用来卸载 .NET Core 的 SDK 和运行时

<!--more-->
<!-- CreateTime:2020/3/12 12:02:44 -->

<!-- 发布 -->

一款好的语言或框架一定要提供好让你放弃时卸载干净的工具

通过官方[github](https://github.com/dotnet/cli-lab/releases/download/1.0.115603/dotnet-core-uninstall-1.0.115603.msi)下载，或我的[csdn](https://download.csdn.net/download/lindexi_gd/12243595)下载

安装之后可以通过下面命令删除干净除了最新版本的SDK和运行时

```csharp
dotnet-core-uninstall remove --all-but-latest
```

如果不是想清理，而是想要放弃了，全部删除，包括 Visual Studio 可能需要的 SDK 请用下面代码

```csharp
dotnet-core-uninstall remove --all --sdk --force
```

更多命令请看 [Microsoft .NET Core 卸载工具](https://docs.microsoft.com/zh-cn/dotnet/core/additional-tools/uninstall-tool?tabs=windows )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

# NuGet 更新库新版本的缓存问题

我有一个 NuGet 库有新的版本，但是我的服务器速度不够快，此时我第一次使用 NuGet 还原找不到库。在我服务器索引完成之后，再次使用 NuGet 会依然找不到这个库，而此时服务器准备完成。这是 NuGet 的缓存的坑

<!--more-->
<!-- CreateTime:2020/10/5 12:38:52 -->

<!-- 发布 -->

我使用了 [BaGet](https://github.com/loic-sharma/BaGet ) 搭建我私有的 NuGet 服务器，他的速度很快，但是索引一个上传的 NuGet 库依然需要一定的时间。如果在 NuGet 服务器还没准备完成之前调用了 NuGet 的 restore 命令，此时预期是找不到 NuGet 的这个新版本的库

但是在 NuGet 服务器准备完成之后，再次调用 NuGet 的还原命令，包括 dotnet restore 都会提示找不到这个版本的库，需要等待超长的时间才能拉

一开始我以为是自己的 NuGet 服务器性能太差，后续在 WPF 官方开源仓库里面学到了这是 NuGet 的坑，在 WPF 的 [Merge WPF 5.0 GA (release/5.0) into WPF master by ryalanms · Pull Request #3605 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3605/files ) 可以看到如下代码

```
Avoid using the http cache as workaround for https://github.com/NuGet/Home/issues/3116
```

在 NuGet 的官方开源仓库有小伙伴提出 [NuGet should refresh its HTTP cache if it can't find the package requested · Issue #3116 · NuGet/Home](https://github.com/NuGet/Home/issues/3116 )

也就是 NuGet 没有刷新自己缓存认为不存在某个库，解决方法是加上 No Cache 命令，如下面代码

```
NuGet restore -NoCache
```

或

```
dotnet restore --no-cache
```

这是因为在 http-cache 文件夹里面没有刷新，关于这个文件夹请看 [How to manage the global packages, cache, temp folders in NuGet](https://docs.microsoft.com/en-us/nuget/consume-packages/managing-the-global-packages-and-cache-folders )

路径如下

- Windows: `%localappdata%\NuGet\v3-cache`
- Mac/Linux: `~/.local/share/NuGet/v3-cache`

可以使用下面代码清理

```
nuget locals http-cache -clear
```

或

```
dotnet nuget locals http-cache --clear
```

使用 No Cache 需要重新下载 NuGet 库，此时的速度比较慢，而使用清理 http-cache 只是刷新版本号的字符串，速度会更快

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

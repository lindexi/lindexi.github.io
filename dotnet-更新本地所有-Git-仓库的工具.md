
# dotnet 更新本地所有 Git 仓库的工具

本文来安利大家一个我做的好用的工具，这个工具可以更新某个文件夹下所有 Git 仓库，通过调用命令行的 Git 命令实现更新。这是一个 dotnet tool 工具，所有代码在 GitHub 完全开源

<!--more-->


<!-- CreateTime:2020/11/3 11:54:13 -->

<!-- 标签: dotnet tool -->
<!-- 发布 -->

我会将一堆 Git 的代码仓库，我将这些代码仓库放在一个大文件夹里面，而我本地的网络比较渣，我期望有一个工具可以辅助我更新这些 Git 代码仓库

我的代码仓库文件夹大概如下，其中 lindexi 文件夹就是存放了一堆代码的大文件夹，而 AsyncWorkerCollection 等文件夹，分别是我在 GitHub 上对应的仓库

```
lindexi
├─AsyncWorkerCollection
├─dotnetCampus.ClrAttachedProperty
├─dotnetCampus.CommandLine
└─WPF
    ├─dotnetCampus.FileDownloader
    └─dotnetCampus.Logger
```

在 Git 命令里面，可以使用如下命令更新代码仓库

```
git fetch --all
```

但是这个代码只能作用在单独一个代码仓库里面，而我的大文件夹里面有大量的 Git 仓库，如果手动一个个去更新，看起来不清真。最清真的方法是通过 dotnet tool 来做

使用 dotnet tool 只有两步，第一步是安装

```
dotnet tool install -g Lindexi.Tool.AutoSyncGitRepo
```

第二步是使用工具

```
AutoSyncGitRepo [文件夹]
```

这个工具可以传入将进行更新的包含 Git 仓库的大文件夹，如果啥都不传将使用当前文件夹

这个工具可以遍历所有子文件夹，也就是放在更低层的 Git 代码仓库也会被找到，进行更新

更新的原理就是命令行调用 `git fetch --all` 进行更新

因此适合作为定时任务调用此工具，这样就可以不断进行更新代码了，在需要使用的代码仓库的时候不需要等待很长的时间去 Git 服务器下载代码

工具的代码在 GitHub 完全开源，请看 [AutoSyncGitRepo GitHub 链接](https://github.com/lindexi/uwp)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
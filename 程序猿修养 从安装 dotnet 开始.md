# 程序猿修养 从安装 dotnet 开始

本来程序员的开始应该是从命令行开始，但是能看到博客的小伙伴，预计都了解命令行了。但是如果是一个空命令行，能做的事情实在不够清真，能提升的效率也有限。如何站在巨人的键盘（没写错）上，用大佬写的各个工具提升日常逗比的效率？答案是从安装 dotnet 开始

<!--more-->
<!-- CreateTime:2020/2/23 19:18:30 -->

<!-- 发布 -->

小伙伴说 dotnet 不就是一个语言框架？其实不然，这是一个总入口，无论想要做什么，其实只要有这个 dotnet 在，就相当于拥有了一堆工具。无论是不是 dotnet 系的开发者，我都推荐你安装 dotnet 这个工具

安装之后有什么好处？其实我可以两句命令行开启一个静态文件服务，提供给手机或其他设备访问本机资源的功能。我可以两句代码开启校验一个文件的 md5 我可以两句代码清理磁盘垃圾

为什么我能做到这些？因为 dotnet 是一个工具的入口，通过 dotnet 可以自己安装工具，然后通过 dotnet 执行工具

好，说到这里，让咱试试

先到 dotnet 官网 [https://dotnet.microsoft.com/](https://dotnet.microsoft.com/) 下载最新版本的 dotnet 工具，然后安装。整个 dotnet 是免费的跨平台的，开源的，也就是想不开可以去随意更改 dotnet 的源代码，构建自己版本的 dotnet 哦

好的，安装完成之后，就来补咱刚才吹的水了，请在命令行敲下这句代码

```csharp
dotnet tool install -g dotnet-serve
```

等等，这是要做什么？其实上面的命令是安装了一个叫 `dotnet serve` 的工具，这个工具可以将本地的文件作为静态文件提供为 web 网页。用命令行进入到一个有文件的文件夹，然后输入下面命令

```csharp
dotnet serve -o
```

此时是不是看到浏览器打开了本地文件的预览了，就是这么简单，一句命令开启静态文件服务器。详细使用请看 [dotnet serve 一句话开启文件服务器 通过 HTTP 将文件共享给其他设备](https://lindexi.gitee.io/post/dotnet-serve-%E4%B8%80%E5%8F%A5%E8%AF%9D%E5%BC%80%E5%90%AF%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8-%E9%80%9A%E8%BF%87-HTTP-%E5%B0%86%E6%96%87%E4%BB%B6%E5%85%B1%E4%BA%AB%E7%BB%99%E5%85%B6%E4%BB%96%E8%AE%BE%E5%A4%87.html)

通过 dotnet 工具，将可以安装世界上超过 100000 个有趣的工具，同时每天都有无数大佬在上传制作自己的工具。在安装 dotnet 之后，除了工具上的便利，还可以提升开发上的便利

在 dotnet 上的工具，除了命令行工具外，还有很多应用软件工具，提供可视化窗口等，如我发布的专业修复 NuGet 合并问题的 NuGet 修复工具，请使用下面命令进行安装和使用

```csharp
dotnet tool install -g NuGetMergeFixTool
dotnet nugetfix
```

上面的安装工具的代码魔力在哪里？其实上面的安装命令需要分为下面这几个部分，第一个是 dotnet 命令，这个是调用刚才安装的 dotnet 工具，而 `dotnet tool` 就是运行 dotnet 里面工具的功能，在这里可以看到这么强大的功能其实只是 dotnet 的一部分。而 `dotnet tool install` 的 install 就是调用安装的功能，除了安装之外肯定还有删除更新等这些功能

接下来的 `-g` 是什么意思？其实这是 `--global` 的缩写，表示安装的是全局的应用，而不是给单个项目用的工具，进行全局安装那么在任何的文件夹里面都能使用上。而最后面的参数就是安装的工具名啦，只需要查阅魔法书，找到工具的名字，就能安装工具，使用强大的功能啦

等等，有一个前提，我的魔法书在哪？其实魔法书存放的地方有很多，也许在你看这个博客的时候，就有小伙伴在创建新的工具啦，而下面是我收藏的一些好用的工具和汇总的网页

- dotnetCampus.UpdateAllDotNetTools 一句命令更新所有 dotnet tool 到最新版本 
- dotnet-serve 一句命令开启文件服务器
- NuGetMergeFixTool 修复大型项目的 NuGet 合并，也可以用来快速升级 NuGet 库

- [natemcmaster/dotnet-tools: A list of tools to extend the .NET Core command line (dotnet)](https://github.com/natemcmaster/dotnet-tools)
- [RehanSaeed/awesome-dotnet-core: A collection of awesome .NET core libraries, tools, frameworks and software](https://github.com/RehanSaeed/awesome-dotnet-core#tools)

如果你有其他的好用的工具还请告诉我

其实这只是 dotnet 工具的一个功能，更强大的在于构建软件和发布的功能。这个功能能让小伙伴从零开始写出一个有趣的软件

程序员从命令行开始

我想要创建一个控制台应用，我可以如何做？通过下面一句命令就可以啦，这就是我在其他设备上，我不需要要求有什么 IDE 或文本编辑工具。只需要用默认的文本编辑工具，如 Windows 下的记事本或 Ubuntu 下的 Vim 都可以，配合 dotnet 工具我就能写出代码咯

```csharp
dotnet new console -o 项目名（如不写将会在当前文件夹里面生成，用当前文件夹名作为项目名）
```

然后如何构建然后运行？没错构建和运行只需要一句代码

```csharp
dotnet run
```

而除了控制台，可以创建的文件也是特别多，如 `dotnet new wpf` 就能创建出 WPF 应用了。接下来就是打包测试发布一条龙的命令啦

```csharp
dotnet test // 自动运行单元测试项目
dotnet pack // 进行打包
dotnet push // 将打出来的包上传到 NuGet 服务器
```

通过不断编写代码开发工具，让工具辅助开发代码，不断循环能够建立起强大的生态。其实现在我开发的时候有很多业务和功能甚至核心功能或算法都不需要从零开始写，只需要去找找有没有大佬做出来了，因为使用 dotnet 的好处在于有 NuGet 的分发方式。小伙伴可以方便将自己写的库通过 NuGet 发布，而其他小伙伴可以通过 NuGet 上去下载库，有一个笑话是发射一颗卫星需要多少步骤？没错就是三步

<!-- 
```csharp
第一步：通过 NuGet 安装发射卫星的库
第二步：调用库里面发射卫星函数
第三步：使用 dotnet run 构建运行程序
``` -->

到底是哪三步呢，还请看下一篇博客 [程序猿修养 使用 NuGet 发射卫星只需要三步](https://blog.lindexi.com/post/%E7%A8%8B%E5%BA%8F%E7%8C%BF%E4%BF%AE%E5%85%BB-%E4%BD%BF%E7%94%A8-NuGet-%E5%8F%91%E5%B0%84%E5%8D%AB%E6%98%9F%E5%8F%AA%E9%9C%80%E8%A6%81%E4%B8%89%E6%AD%A5.html)

安装完成了 dotnet 工具之后，将让设备具备下载和使用海量的工具，这些工具不仅仅只是命令行工具，还有很多应用软件在内。还可以让设备具备构建和发布软件的功能，和让设备分发工具和库的功能

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

# WPF 框架开发 调试和开发 System.Xaml 的独立项目方法

本文将给大家一个 System.Xaml 的独立项目，此项目代码和 WPF 仓库的 System.Xaml 项目相同，但本文的 System.Xaml 的独立项目不依赖 WPF 其他项目，构建方便，构建速度快，搭配 System.Xaml.Demo 项目更方便调试


<!--more-->

<!-- 发布 -->
<!-- 标签：WPF,XAML -->

本文是对新入手开发 WPF 框架的开发者友好的，只要对 WPF 有一些了解即可入手。本文的 WPF 框架开发，指的是开发 WPF 这个框架，开发 WPF 本身，而不是开发基于 WPF 框架的应用

在 WPF 中，所有编写的 XAML 代码，最终都会依赖 System.Xaml 库进行执行。也因此 System.Xaml 就是 WPF 的 XAML 的核心入口。但 System.Xaml 从命名上可以看到，这是和 WPF 框架没有强相关的库，这是一个专门用来处理 XAML 相关的库

也因为 System.Xaml 库是很独立的，因此 WPF 官方开发者 [Ryland](https://github.com/ryalanms) 想要独立分发 System.Xaml 库，请看 [Consider shipping System.Xaml as a separate netstandard package · Issue #4140 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4140 ) 和 [System.Xaml platform independence · Issue #3543 · dotnet/wpf](https://github.com/dotnet/wpf/issues/3543 )

本文的实现就是将 System.Xaml 库独立，干掉所有对 WPF 仓库依赖，放在我的 GitHub 仓库里面。只需要从 GitHub 拉下我的代码，即可获取独立的 System.Xaml 库，此时的构建和调试都比放在 WPF 仓库方便。当然缺点是没有更上 WPF 的源代码版本，需要大家自己手动去拷贝最新的代码

以下是获取和构建的方法

在开始之前，先确定你已经安装了 VisualStudio 2019 或 VisualStudio 2022 或更高版本，以及 Git 命令行工具

先从 GitHub 或 Gitee 使用以下命令行拉下代码，请打开你的本机命令行工具，如在运行窗口输入 cmd 即可打开命令行窗口。当然，如果连这一步都不知道如何做的，那还是劝退吧

在命令行窗口，先使用 cd 命令进入某个文件夹，这个文件夹将会用来存放 System.Xaml 的独立项目代码

```
cd 某个用来存放 System.Xaml 的独立项目代码的文件夹
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cae701f994ad396429303e7ec249b9ee4a693839
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cae701f994ad396429303e7ec249b9ee4a693839
```

获取代码之后，进入 System.Xaml.Test 文件夹

打开 `System.Xaml.Test\System.Xaml.Demo\System.Xaml.Demo.sln` 即可看到 Demo 和 System.Xaml 库，尝试设置 System.Xaml.Demo 作为启动项目，然后按下 F5 即可构建运行

欢迎大家参与 WPF 框架的开发

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

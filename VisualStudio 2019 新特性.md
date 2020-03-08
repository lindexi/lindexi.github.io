# VisualStudio 2019 新特性

很多小伙伴都好奇 VisualStudio 2019 有哪些功能，下面让我介绍一些好玩的特性

<!--more-->
<!-- CreateTime:2019/10/31 8:48:27 -->

<!-- 标签：VisualStudio -->

在安装完成之后会看到创新的欢迎界面，这个欢迎界面支持输入关键字搜项目，同时支持选择语言平台

![](https://i.loli.net/2019/04/01/5ca1f8b2b8420.gif)

很多小伙伴都说 VisualStudio 卡，于是微软就做了项目性能的提升，特别是打开大的项目

在新的 VisualStudio 支持打开项目的时候不加载项目

![](https://i.loli.net/2019/04/01/5ca1f90b01cb2.jpg)

打开的时候就发现所有的项目都没有加载，但是这样就需要手动加载一些项目才可以开始。但是下一次打开是不是还需要打开不加载项目，然后手动加载需要的项目？其实有这个功能，支持保存解决方案加载的项目，也就是假如我有10个项目，我需要在启动的过程不加载其中的5个单元测试项目，就可以通过这个方法另存 slnf 文件，下次打开这个文件就会加载需要的项目，详细请看[VisualStudio 解决方案筛选器 slnf 文件](https://blog.lindexi.com/post/visualstudio-%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E7%AD%9B%E9%80%89%E5%99%A8-slnf-%E6%96%87%E4%BB%B6 )

![](http://image.acmx.xyz/lindexi%2F201941194410257)

在设置里面默认打开了允许并行项目初始化，如果打开大项目就会发现整个 CPU 和硬盘都在被 VisualStudio 使用，这样可以做到很快的加载

![](http://image.acmx.xyz/lindexi%2F201941194516172)

在 VisualStudio 经过用户的上报发现用户用的最多的是 Blue 主题，同时也只有对 Blue 主题做优化才有产出比，如果是深色主题，无论怎么优化的对比度都不如优化 Blue 的，于是微软就对 Blue 主题做了优化，添加了额外对比度，实际上我看不出差别

![](http://image.acmx.xyz/lindexi%2F20194119465644)

然后对代码还有新的高亮

![](http://image.acmx.xyz/lindexi%2F201941194638185)

在 Windows10 会启动 Per-Monitor Awareness 提高清晰

![](http://image.acmx.xyz/lindexi%2F20194119471192)

在小伙伴对 VisualStudio 的另一个吐槽是更新太慢，原因是 VisualStudio 更新需要下载很多的文件，所以不到不需要使用的时候才不更新。

在新 VisualStudio 支持先自动下载文件，然后提示用户安装，这样可以减少安装时间。这个功能对于在国内的小伙伴十分好，因为我的网速大概是 100K 也就是下载一个更新需要半天

因为 VisualStudio 主要功能在于写代码，需要将更多的空间给代码，在 VisualStudio 2019 将标题栏放在了下一行，这样可以让代码可以使用的空间多了标题栏

![](http://image.acmx.xyz/lindexi%2F201941195235982)

可以发现 VisualStudio 2019 带来一波很有用的性能提示，但是也有另一些没有什么用的性能提升，如性能管理器

![](http://image.acmx.xyz/lindexi%2F201941195333378)

打开这个界面可以看到基本只有一个不能卸载的插件是降低性能

![](http://image.acmx.xyz/lindexi%2F201941195359275)

现在的微软的策略是开源，于是需要更多社区的支持

新的格式支持一键打包

但是打包存在一个问题是许可证，可以设置表达式或文件，表达式就是如 Apache 协议，文件就是放一个文件在库，这样可以用自己的合同，如使用 996 协议

![](http://image.acmx.xyz/lindexi%2F201941195540823)

在社区的支持另一个容易让小伙伴打起来的是格式化，于是微软弄了开源项目[dotnet format](https://github.com/dotnet/format) 在不同的工具使用相同的格式化

![](http://image.acmx.xyz/lindexi%2F201941195654423)

虽然有格式化工具但是小伙伴还是会觉得别人的格式化不对，于是这个工具还支持小伙伴在自己的仓库使用私有的 editor config 配置

使用 VisualStudio 最主要的功能有两个，一个是编辑功能，一个是调试功能

编辑工具现在有很多原有的 Resharper 的功能，如界面可以提示当前打开的文件存在哪些可以优化的代码

![](http://image.acmx.xyz/lindexi%2F201941195919863)

可以点击按钮转到下一个提示

![](http://image.acmx.xyz/lindexi%2F201941195919863)

还在界面添加了清理代码的功能，如果用过 Resharper 的小伙伴就发现，在 Resharper 有两个格式化的方法，一个是 ctrl+alt+enter 格式，另一个是 clean file 格式，如果格式化的功能多了，那么格式化的速度就会比较慢。所以 VisualStudio 提供两个格式化选择，默认的第一个就是简单的格式化，同时支持配置格式化包括的修补

![](http://image.acmx.xyz/lindexi%2F2019412031922)

在写代码的时候，如果打一个字要等半天是不是会砸键盘，在 VisualStudio 2019 可以设置插件可以拖慢的时间，如果超过时间就会杀线程，也就是这个功能可以看到 Resharper 停止工作

![](http://image.acmx.xyz/lindexi%2F20194120512942)

另一个主要功能调试也有了一些优化，在自动窗口、局部变量窗口、监视都支持搜变量和内容，同时对于对象树可以设置深度

![](http://image.acmx.xyz/lindexi%2F20194120721367)

在 VisualStudio 还有一个很好用，但是很少有小伙伴知道的功能是格式化变量显示

![](https://i.loli.net/2019/04/01/5ca1ff625e2e4.gif)

在调试性能工具细化了功能，对于 VisualStudio 2017 添加了两个功能，还有热路径

![](http://image.acmx.xyz/lindexi%2F20194120119454)

点击热路径就会显示建议优化的调用

![](http://image.acmx.xyz/lindexi%2F201941201136511)

对语言 与 .NET Core 也有很多优化，特别是 .NET Core 桌面开发

![](http://image.acmx.xyz/lindexi%2F201941201246899)

还有很多有趣的功能，就需要小伙伴下载安装

<!-- ![](http://image.acmx.xyz/lindexi%2F20194120133155) -->

![](http://image.acmx.xyz/lindexi%2F20194310939421)

其实还有很多我没有说到的细节，此时请大家看大佬们的视频 [Visual Studio 2019 新特性大揭秘](https://devopslive.bopoda.cn/live/azuredevops101-20190403?from=groupmessage&isappinstalled=0 )

特别感谢

- 应颜小伙伴告诉我 VisualStudio 写错了

- walterlv 告诉我为什么优化的是 Blue 主题

- Edi Wang 小伙伴让我撤回容易被小伙伴喷的图

如何下载安装离线的 VS 请看 [VisualStudio 2019 如何离线下载](https://blog.lindexi.com/post/visualstudio-2019-%E5%A6%82%E4%BD%95%E7%A6%BB%E7%BA%BF%E4%B8%8B%E8%BD%BD )

附激活码，激活码只用于尝试 VisualStudio 的使用，请不要在商业环境使用

Visual Studio 2019 Enterprise

BF8Y8-GN2QH-T84XB-QVY3B-RC4DF

Visual Studio 2019 Professional

NYWVH-HT4XC-R2WYW-9Y3CM-X4V3Y

[Visual Studio 2019 .NET productivity](https://devblogs.microsoft.com/dotnet/visual-studio-2019-net-productivity-2/ )

<!-- 过滤引用 解决方法 值断点 -->

[VisualStudio 2019 尝试使用 C# 8.0 新的方式](https://blog.lindexi.com/post/VisualStudio-2019-%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8-C-8.0-%E6%96%B0%E7%9A%84%E6%96%B9%E5%BC%8F.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

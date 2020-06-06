# C# dotnet 本地代码构建没问题，但 CI 自动构建失败可能的原因

本地构建能通过至少代码上的问题不大，本文列举了一些可能的原因，小伙伴可以按照顺序依次查看代码和配置

<!--more-->
<!-- CreateTime:6/5/2020 8:37:32 AM -->

<!-- 发布 -->

## 代码分支

默认的 CI 和 CD 构建的分支应该是主分支或开发分支，而此时服务器构建的代码的版本也许和本地调试的代码的分支不相同

例如服务器上的运行 CI CD 的 git 仓库分支是 master 分支，而本地开发使用的是个人分支 `t/lindexi/doubi` 而此时有另一个小伙伴更改了 A 文件的代码，在代码里面更改了 API 接口

而我在 B 文件开发，调用了原本 A 文件的原本的接口，此时一定我本地是能构建通过的，而我本地的这个分支在服务器上 CI 也是能通过的。但是在合并到 master 之后进行 CI CD 就会炸了

因此，如果发现服务器 CI CD 构建失败了，请尝试拉取服务器的构建分支，如 master 分支，尝试在本地构建

## NuGet 源

基本上只会在项目刚配置的时候翻车，如果本地使用了私有的 NuGet 源，而服务器没有这个源，那么将拉包失败，构建不通过

因此判断是否此问题应该想看构建输出日志，如果是服务器报告说还原 NuGet 失败，找不到 xx 包。那么第一可能是 nuget.org 连接不上，第二可能就是本地使用了私有源没有配上服务器

第一个问题的解决方法一般只有等咯。但是在等的过程不妨看看是否是第二个可能，本地实际上使用了私有源

自己搭建一个 NuGet 服务器是十分简单的，我使用了 [BaGet](https://github.com/loic-sharma/BaGet) 搭建了一个私有源，我本地全局配置了这个私有源。但是服务器上使用 docker 构建，而 docker 每次都是全新的环境，除非做这个构建镜像的时候加上了配置，否则服务器上是没有全局配置的。因此服务器找不到放在私有源的 NuGet 包，服务器就拉不到包，也就构建失败了

这个问题解决方案很简单，就是项目级配置用到的私有源，配置方法请看 [VisualStudio 给项目添加特殊的 Nuget 的链接](https://blog.lindexi.com/post/VisualStudio-%E7%BB%99%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E7%89%B9%E6%AE%8A%E7%9A%84-Nuget-%E7%9A%84%E9%93%BE%E6%8E%A5.html )

用 docker 做构建服务器的一个不足是没有了 NuGet 的本地缓存，每次都需要访问服务器，因此有一个内网的私有服务器还是能提升一些效率

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

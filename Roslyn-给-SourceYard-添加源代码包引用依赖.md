
# Roslyn 给 SourceYard 添加源代码包引用依赖

小伙伴告诉我，使用 SourceYard 制作的源代码无法传递源代码包的引用依赖。也就是我如果源代码包 A 引用了源代码包 B 而我使用项目 C 安装了源代码包 A 的时候就不会自动安装源代码包 B 的依赖。本文来告诉大家如何修复这个问题

<!--more-->


<!-- CreateTime:6/16/2020 8:25:22 AM -->

<!-- 不发布 -->

在 [https://github.com/dotnet-campus/SourceYard/issues/60](https://github.com/dotnet-campus/SourceYard/issues/60) 有小伙伴就报告了这个问题，这个问题不是阻断的问题，只是在使用上有点坑

默认的 NuGet 包都会添加引用依赖的传递，作用是让小伙伴使用的时候比较简单。例如我写了一个库 A 这个库想要运行需要依赖安装了 B 库

此时我期望小伙伴通过 NuGet 安装了我的 A 库的时候，自动通过引用依赖安装上了 B 库。这样就能简化小伙伴的开发，直接安装一个库就能运行

但是 SourceYard 制作的源代码就没有这个功能

那么什么是 SourceYard 呢？咱的 NuGet 默认发布的是 dll 文件，也就是二进制文件，而 SourceYard 能打出来源代码包，源代码包里面分发的是源代码而不是 dll 文件。作用是能做到二进制的兼容，同时减少最终输出项目的程序集，提升调试库的效率

那为什么 SourceYard 打出来的源代码不会做到传递依赖？原因是如果我在 A 库里面引用了一个我不期望被传递依赖的库，例如某个代码检查工具包，此时我需要设置这个工具包是  PrivateAssets 为 all 表示不要引用传递

而对于安装的源代码包来说也是需要设置这个值的，例如我在 A 库里面安装了源代码包 B 库，此时我如果不是将 A 库作为源代码包分发，而是新建一个 C 项目直接引用 A 库。如果此时的 B 库没有表示为私有引用，那么将会在 C 项目因为依赖传递自动安装了 B 库。而小伙伴都知道，现在 A 库构建的程序集已经包含了 B 库的代码了，而此时的 C 库如果也安装了 B 库，那么将会出现源代码的重复

因此默认的 SourceYard 的源代码包是设置不要传递依赖的，也是这个原因是上面的 A 库打出源代码包的时候没有添加 B 库的引用依赖

那么这个问题可以如何解决呢？简单的方法是判断在将 A 库打出源代码包的时候将找到 A 库安装了哪些源代码包，将这些源代码包添加到 NuGet 依赖就可以了

详细请看视频

[Roslyn 给 SourceYard 添加源代码包引用依赖 - 哔哩哔哩](https://www.bilibili.com/read/cv6421416)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
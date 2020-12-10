# 苏州 开源自主的 dotnet 生态

本文仅仅只是用来在2020苏州开发者大会上的素材

<!--more-->
<!-- CreateTime:2020/12/10 19:15:52 -->

<!-- 草稿 -->

从编程语言版权，到编译器到运行时的开源，和允许自己分发和构建

从咱写下的 C# 代码，到在机器上运行的应用，有多少个步骤是咱可以自主的

在 dotnet 里面，将使用开源的基于 MIT 协议的 Roslyn 编译器，将 C# 代码编译为 IL 文件。在执行 dotnet 应用的时候，将会通过开源的 dotnet runtime 运行时，将 IL 代码转换为机器码执行。而基于 dotnet 的上层应用的框架，如 WPF 和 WinForms 和 ASP.NET Core 等都是基于 MIT 协议开源的

在开源的世界里面，采用 MIT 协议是最友好的，意味着使用方无任何版权费用问题，意味着可以对整个开源仓库进行自由的更改

说到对开源仓库的更改，不得不提一下，是否改的动。当然了答案肯定是可以的

从 Roslyn 编译器开始，整个 Roslyn 编译器是采用 C# 编写的，依托于 dotnet 运行时运行。在 GitHub 上不仅开放了所有源代码，同时还开放了整个构建工具链，只需要通过 Build.cmd 脚本就能构建整个仓库，输出可以用来构建 Roslyn 编译器和 dotnet runtime 的 Roslyn 编译器。这是一个有趣的套娃，咱将使用 Roslyn 编译器来构建 Roslyn 编译器。咱将使用 Roslyn 编译器来构建 dotnet runtime 用来运行 Roslyn 编译器

整个 dotnet runtime 运行时同样在 GitHub 上完全开源，在 dotnet 5 的时代进入了大一统，将 Mono 等全部收到 dotnet runtime 仓库。整个 dotnet runtime 也开放了整个构建工具链，只需简单的运行 build.cmd 脚本即可完成整个 dotnet runtime 的构建

在 https://www.bilibili.com/video/BV1e541157CB/ 的视频就是自己构建 Roslyn 编译器，然后用构建出来的 Roslyn 编译器构建运行时




使用 Roslyn 编译器构建出来的输出是一个 IL 文件，在具体的平台上，需要有不同的启动器。启动器的作用就是让 dotnet 运行时跑起来，启动器需要对不同的平台构建不同的版本，作为应用的入口。在启动了运行时之后，将进行 JIT 解析执行 IL 内容，执行应用的逻辑。启动部分请看 [dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87%E8%87%AA%E5%B7%B1%E5%86%99%E4%B8%80%E4%B8%AA-dotnet-host-%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

如此能做到让 IL 文件做到平台无关，可以轻松跨平台





整个 dotnet runtime 除了提供了让 dotnet 应用运行的基础，包括了 IL 即时编译，以及 GC 内存管理等之外，还包含了大量的基础库



说明生态活跃度：

从 Roslyn 编译器到 dotnet runtime 仓库都是在 dotnet 基金会组织下，有大量的来自世界各地的第三方开发人员参与了开发。此时可以加上 dotnet runtime 仓库的截图

现在有大量的上层应用框架都选择了开源贡献，如整个国内的 dotnet 组织 https://github.com/dotnetcore/ 和老张的包含了一些新鲜的，不错的，但没推广无人知晓的框架的 https://github.com/BaseCoreVueProject 组织。 各地散落的高质量开源仓库，如在 Gitee 上有近 2k 星的 Fursion 框架等。更多需要收藏，请看 [开源项目](https://blog.lindexi.com/post/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE.html )



说了这么多，其实不如自己动手构建一次，建立起来的信心。构建 Tip 请看

土豪的微软提供了 GitHub Action 免费的构建服务器可以使用，如果发现自己在国内因为工具链需要大量的下载内容而无法构建成功，可以使用 GitHub 的 Action 来进行构建

构建私有版本的 WPF 仓库请看 [手把手教你构建 WPF 框架的私有版本](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%9E%84%E5%BB%BA-WPF-%E6%A1%86%E6%9E%B6%E7%9A%84%E7%A7%81%E6%9C%89%E7%89%88%E6%9C%AC.html )








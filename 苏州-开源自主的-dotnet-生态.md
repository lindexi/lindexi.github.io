
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




相信今年大家都听过一个很火的词，自主可控【转场用】。咱接下来聊聊 dotnet 在开源的自主可控方面。从咱写下的 C# 代码，到在机器上运行的应用，有多少个步骤是咱可以自主的？【停顿一下，聚拢一下注意力】尽管答案大家都知道，那就是完全开源完全可控。但我依然想要用一个简单的例子来告诉大家究竟有多开源多可控

想必在座的各位都写过 Hello World 吧【重新开场 停顿】。从写下 Hello World 的 C# 代码到控制台打印出来，咱用到了 dotnet 里面多少的技术？ 现在是 2020 年，几乎所有的高级语言都需要经过编译才能运行，咱的 C# 也不例外。在 dotnet 的世界里面，咱将会用到开源的基于 MIT 协议的 Roslyn 编译器，将 C# 代码编译为 IL 文件。【停顿】这个 Roslyn 编译器很有意思，因为这是一个一半用 C# 一半用 VB 写的编译器，这是一个多么有趣的套娃游戏：用 C# 写的 Roslyn 编译器来编译 C# 代码。我特别推荐大家将 Roslyn 开源仓库拉下来，自己构建一下，用自己构建出来的 Roslyn 编译器来编译自己的代码。这样做过一遍之后，相信大家将会对整个 dotnet 的开源体系有更深的理解。在 2020 年，其实自己基于开源的 Roslyn 定制一个属于自己的编译器没有想象中那么难，而 Roslyn 编译器的开源协议是最友好的 MIT 协议，这意味着咱可以对整个开源仓库进行自由的更改

（这将允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。唯一的限制是，软件中必须包含上述版 权和许可提示，后者协议将会除了为用户提供版权许可之外，还有专利许可，并且授权是免费，无排他性的(任何个人和企业都能获得授权)并且永久不可撤销，用户使用完全不用担心收费问题和版权问题，以及后续无法维护问题）

（以上内容的细节请看 [dotnet 从 Roslyn 编译器到 dotnet runtime 运行时的构建_哔哩哔哩 (゜-゜)つロ 干杯~-bilibili](https://www.bilibili.com/video/BV1e541157CB )）

在使用 Roslyn 编译器编译 C# 代码之后，如果咱不耍什么如 dotnet Native 等黑科技，那咱编译 C# 代码的输出文件将会是一个包含 IL 的 DLL 文件。此时也许大家有点疑惑，那我控制台编译出来的 EXE 文件又是什么呢【停顿，有认知冲突，用于聚拢一下注意力】。其实这个 EXE 里面不包含咱的 IL 代码（单文件除外哈）这个 EXE 只是一个 Native 的启动器而已，这个文件的内容就是传统的 Win32 应用的逻辑，用的是 Native (汇编)二进制内容，用途是将 dotnet runtime 跑起来。为什么需要这样一个启动器呢，其中一个原因是为了更好的跨平台。每个平台都有自己定义的可执行文件格式，为了让 dotnet 有更好的适配性（不要去碰具体平台的意思），咱需要在具体的平台上，需要有不同的启动器。启动器的作用就是让 dotnet 运行时跑起来，启动器需要对不同的平台构建不同的版本，作为应用的入口。在启动了运行时之后，将进行 JIT 解析执行 IL 内容，执行应用的逻辑。这样做能让 IL 文件做到平台无关，可以轻松跨平台，换句话说，我在 Linux 上使用的 DLL 和在 Windows 上使用的DLL可以是完全相同的一个文件

（以上内容的一个细节请看 [C# dotnet 从代码到程序运行过程发生了什么_哔哩哔哩 (゜-゜)つロ 干杯~-bilibili](https://www.bilibili.com/video/BV1cf4y1e7iC )）
（博客请看 [dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87%E8%87%AA%E5%B7%B1%E5%86%99%E4%B8%80%E4%B8%AA-dotnet-host-%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )）

咱刚才提到了 dotnet runtime 运行时，这也是在 GitHub 上完全开源的，基于最友好的 MIT 开源协议的大仓库。为什么说是大仓库呢？因为这个 GitHub 仓库里面将原本散落在各地的 dotnet 仓库集成到一起，这也就包括了 Mono 等仓库。在这个仓库里面将包含了 dotnet 程序运行过程中，将会用到的基础模块。如 JIT 将 IL 转换为本机代码，执行 GC 用于内存管理（看看伟民哥来不来个插入广告了，“值得一说的是，在 dotnet 里面，内存管理是一个复杂而有趣的话题，伟民翻译了一本 .NET内存管理宝典 的书，里面有对此详细的介绍”）。除此之外还包含了咱日常使用的大量底层基础库，如 System.Console 等。这些基础模块基本上都是由 C# 编写，因为这样的对某个代码的优化将会全平台受益。有趣的是，咱的 Roslyn 编译器的运行也是需要用到 dotnet runtime 的，也需要用到很多基础库，因此这就有一个有趣的套娃了，咱使用 C# 写的 Roslyn 编译器来编译 C# 代码，用运行在 dotnet runtime 上的 Roslyn 编译器来编译 dotnet runtime 仓库。这就是为什么我一开始推荐大家自己去编译一次 Roslyn 编译器的原因了，这就是一切的开始的入口。那如何才能构建呢【根据表现，用来提升注意力】？其实在 dotnet 开源组织的各大仓库里面，包括 Roslyn 和 dotnet runtime 仓库，开源的不仅是代码本身，还有构建这个仓库整个工具链，因此只需要在网络速度足够好的设备上，双击一下 Build.cmd 即可完成构建。也许网络速度足够好这个词限制了一些朋友，不过没关系，土豪的微软收购了 GitHub 提供了 GitHub Action 免费的构建服务器可以使用，如果发现自己在国内因为工具链需要大量的下载内容而无法构建成功，可以使用 GitHub 的 Action 来进行构建

还有什么比自己实实在在构建一次更能说明说明自主可控的【这句话看起来不好】

咱刚才提到的无论是 Roslyn 编译器还是 dotnet runtime 仓库，都是在 dotnet 基金会旗下的。而 .NET 基金会2014年微软组织成立的一个独立的组织。2014年以来已经有众多知名公司加入.NET基金会, 仅在平台项目中，.NET平台上有大量贡献者其实不在Microsoft工作。而 dotnet 基金会旗下包含了 dotnet 体系下的各个应用层框架，如 WPF WinForms ASP.NET Core Blazor 以及下一代 UI 框架 MAUI 等等，这些仓库都是完全开放的，欢迎大家参与开发




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
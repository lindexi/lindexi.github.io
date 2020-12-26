# WPF 最简方法使用自己定制的 WPF 框架

本文提供了一个最简的方法，可以用到整个 WPF 框架里面所有 internal 内部权限的成员的方法。这是一个我自己定制的 WPF 框架，可以在此基础上构建属于自己的定制化的 WPF 框架

<!--more-->

<!-- 发布 -->

本文提供的方法适用于 .NET 5 和 x86 下，如果需要其他版本，请自行构建和使用，关于如何自行构建和定制化，请看 [手把手教你构建 WPF 框架的私有版本](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%9E%84%E5%BB%BA-WPF-%E6%A1%86%E6%9E%B6%E7%9A%84%E7%A7%81%E6%9C%89%E7%89%88%E6%9C%AC.html )

现在的 WPF 属于 dotnet 基金会组织下的一个开源仓库，基于 MIT 协议，意味着我可以进行魔改然后私有发布甚至不再公开源代码商业使用。而 WPF 是一个跨了很多代技术的框架，在 WPF 仓库里面既可以看到最新的 .NET 5 的代码也可以看到上古的 Perl 的构建代码。这样就会存在一个问题，本地构建难度很高，调试难度也很高。想要在本地构建出来，需要自己的设备的网络能访问世界上任意的网络。而如果想要在 VisualStudio 上直接构建出来，那就需要很多 msbuild 的预编译知识，和一些引用关联知识，虽然不难，但是知识量还是摆在这里的

于是此时的给 WPF 框架开发的劝退力就太足了，想要自己定制化属于自己的 WPF 框架难度有点大，至少我每次定制化的步骤都有点多。因此我就在摸索最佳实践，找到了最简的方法，请看下文

先不聊啥原理，原理部分我将会放在其他的博客里面，原因是原理太多了

使用本文的方法能实现的是，在我提供的程序集里面，可以访问到 WPF 框架里面所有 internal 内部权限的成员。限制是只适用于 .NET 5 和 x86 下，如需有其他定制还请阅读我的原理博客，或者找我私聊让我帮你做一些基础搭建

在开始之前，需要说明的是，即使是最简方法，依然还是有一些小限制的。首先你需要本地有安装 7z 压缩工具，其次你需要一个包含了最新版本的 VisualStudio 神器

准备好了之后，咱就开始吧

第一步是下载我提供的基础框架，可以从 [CSDN 下载](https://download.csdn.net/download/lindexi_gd/13769715) 或者给我一封邮件让我发给你

第二步是解压缩下载的基础框架，下载下来是一个 7z 压缩包，需要解压缩才能使用

第三步就是双击 dotnetCampus.WPF.sln 打开

好了，贺喜你，基础部分就完成了，接下来就只需要在 Program.cs 写代码就可以了。在 dotnetCampus.WPF 程序集里面写的代码，能用到 WPF 框架里面所有 internal 内部权限的成员

在 Program.cs 代码里面可以看到我的模版内容

```csharp
        [STAThread]
        static void Main(string[] args)
        {
            var application = new Application();
            var window = new Window()
            {
                Title = "林德熙是逗比"
            };
            window.Loaded += (sender, eventArgs) =>
            {
                // 这里的 GetAppWindow 是 internal 的方法，但是在这个程序集可以访问
                var navigationWindow = application.GetAppWindow();
            };
            application.Run(window);
        }
```

如果你尝试在自己的 WPF 项目里面，调用 Application 类的 GetAppWindow 方法，将会提示你没有访问权限或者没有这个方法。但是在这个程序集里面，你可以随意的访问这些 internal 方法

其实 WPF 框架的设计在大体上是十分好的，大部分的定制都能通过调用 itnernal 内部权限成员，如类或方法实现。有了这个基础框架，就能极大提升开发的 WPF 框架的效率，将大部分的实现逻辑放在 dotnetCampus.WPF 程序集。这样能提供非常简单的本地构建调试方法，就和其他基础项目相同的构建调试方法，不需要去了解 WPF 框架相关构建知识

我推荐使用此方法来进行一部分不更改 WPF 已有逻辑的开发，而事实上作为一个庞大的框架，官方 WPF 团队其实也不敢合入对已有逻辑有比较多更改的内容，谁知道有哪个模块静默依赖了。因此更多的是新加，如新加某些新的类或者多开放某些方法等等。当然了即使是不合入官方仓库，自己用的，我也不推荐更改已有的逻辑，因为大家也很难测试全。因此在不更改已有的逻辑下，使用此方法开发的效果和在 WPF 项目里面更改的效果几乎相同

用这个方法构建出来的应用就是用上了自己提供的定制的 WPF 框架，还请大家试试使用此方法

本文提供的这个程序集的最佳实践方法就是提供转换器，传入某个 WPF 框架的类，接着在这个程序集里面使用代理模式这个设计模式将这个类里面的内容开放出来或者进行定制

为什么会选用本文的这个方式来作为最简方法？原因是如果使用 VisualStudio 进行编辑 WPF 框架，会遇到这样的问题，更改一点东西就需要来一个构建，而构建 WPF 是一个缓慢的过程。因为我的技术不够还做不到让 WPF 框架支持增量编译，因此每次构建最少都是 20 分钟。这样的调试效率太低了

实际上本文提供的方法是给 WPF 每个项目都加上了一句 InternalsVisibleTo 到 dotnetCampus.WPF 程序集而已

使用这个方法也是有缺点的，如需要额外添加整个程序集以及在 dotnetCampus.WPF 程序集里面是不能放 xaml 文件的

我还同步将内容放在 [最简方式定制 WPF 框架_哔哩哔哩 (゜-゜)つロ 干杯~-bilibili](https://www.bilibili.com/video/BV1hh411f7jH )

具体实现请看 [WPF 框架开发 加入 InternalsVisibleToAttribute 特性让其他程序集可以访问 internal 权限成员](https://blog.lindexi.com/post/WPF-%E6%A1%86%E6%9E%B6%E5%BC%80%E5%8F%91-%E5%8A%A0%E5%85%A5-InternalsVisibleToAttribute-%E7%89%B9%E6%80%A7%E8%AE%A9%E5%85%B6%E4%BB%96%E7%A8%8B%E5%BA%8F%E9%9B%86%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE-internal-%E6%9D%83%E9%99%90%E6%88%90%E5%91%98.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

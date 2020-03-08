# WPF 使用 MyScript 的 IInk 做手写识别

有小伙伴问我可以如何在 WPF 使用其他第三方提供的库进行手写识别，上次 MyScript 的工程师和我吹，他做了世界上识别最好的库，本文就来安利一下大家这个库。这里库是收费的库，但是可以免费使用，只要不是有大量用户，这个库还是免费用的。用这个库可以在 Windows 平台识别数字、多个不同语言、数学公式手写识别

<!--more-->
<!-- CreateTime:2019/11/20 8:18:26 -->

<!-- csdn -->

我对比了这个库和微软自带的手写识别，在识别英文和数字上，其实差别不大。但是 MyScript 的 IInk 对数学公式的识别是超级厉害，至少我还没有找到一个比他厉害的。但是在识别中文上就比微软自带的差一点了

在 MyScript 的使用限制是需要指定当前用户书写的内容，如先设置为英文，此时就会使用英文识别。另外识别的计算速度不快，不过做实时识别还是可以的，这要求你的设备不能太差。基本上能将 LoL 运行起来，玩的时候不会砸键盘的设备就能做实时识别

好了，我知道怎么吹小伙伴都是不信的，让我给出一张图片证明。这是一张比较大的 GIF 图片，所以我放在了 sm.ms 上，如果看不到图片就刷新

![](https://i.loli.net/2019/11/18/EH9TIqwUndyBVft.gif)

小伙伴也许会认为我是用了100000挑里面最好的一个放上来，下面就来告诉搭建如何开发

不要问那么多，上车咯

先通过 Github 下载官方的例子 [MyScript/interactive-ink-examples-wpf: MyScript Interactive Ink examples and user interface reference implementation for Windows WPF.](https://github.com/MyScript/interactive-ink-examples-wpf )

下载完成之后，不要急着打开，因为这是一个收费的库，所以需要下载许可证。下载许可证的路线有点长，需要注册帐号写申请，然后这个网站又在国外，所以可能需要小伙伴一点时间。申请帐号地址是 [https://developer.myscript.com/getting-started](https://developer.myscript.com/getting-started) 申请完成会在邮件收到 MyCertificate.cs 文件，将这个文件替换到下载的仓库的 `GetStarted\MyCertificate.cs` 和 `Demo\MyCertificate.cs` 如果小伙伴都不想申请，或者打不开网站，可以发邮件[给我](mailto:lindexi_gd@163.com)使用我的文件

免费的 MyCertificate.cs 可以用在100个设备上

如果你看到本文时，官方还没有修复 `MyScript.InteractiveInk.GetStarted.Wpf-VS2017.csproj` 的坑，也就是我在 [github](https://github.com/MyScript/interactive-ink-examples-wpf/issues/2) 上提出的问题，那么请小伙伴手动删除 `<HintPath>..\..\..\..\..\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.6.1\PresentationCore.dll</HintPath> ` 这段代码，以及下面差不多的代码

此时通过 VisualStudio 打开，设置 MyScript.InteractiveInk.GetStarted.Wpf-VS2017 作为启动项目，还原 NuGet 库，然后就可以运行了

此时运行如何编译太久了，请看仓库的 recognition-assets 文件夹是否有内容，在 `getRecognitionAssets.ps1` 将会下载一些需要的库，可以自己手动下载替换文件。如果你无法下载，请发邮件[给我](mailto:lindexi_gd@163.com)给你文件

运行默认的是文字识别，可以识别英文和数字，可以做到实时识别

如果想要用本文说的手写公式识别功能，请打开 `GetStarted\MainWindow.xaml.cs` 文件，修改 `PART_TYPE` 的值

```csharp
        private const string PART_TYPE = "Drawing";
      
```

修改之后请重启程序，现在就可以试试公式识别，需要手动点击 Convert 按钮才能识别

这个库好不好用，请小伙伴自己测试如果还是无法编译成功，请找我要编译完成文件

再次说明，我没有收 MyScript 的好处

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

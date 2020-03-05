# 调试 ms 源代码

如果需要调试 WPF 源代码或框架源代码，那么需要使用 DotPeek。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->

<!-- 标签：调试，dotpeek,反编译 -->

首先需要下载 dotPeek ，可以到官网下载 [dotPeek: Free .NET Decompiler & Assembly Browser by JetBrains](https://www.jetbrains.com/decompiler/) 还可以到 csdn [下载](http://download.csdn.net/download/lindexi_gd/10133189 )

首先打开 dotPeek 然后点击启动符号服务器，所有符号。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201798184736.jpg)

然后点击工具设置，可以看到这个页面

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201798185154.jpg)

然后打开 VS 工具选项，在调试设置符号，刚才已经复制了，现在添加就好

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20179819559.jpg)

然后还需要去掉微软的服务和本地缓存

然后写一个呆磨进行测试

现在就可以开始调试框架源代码了

只需要在一些函数使用断点，然后堆栈跳转，假如我在 MouseDown 写一个断点，在触发按下，点击堆栈，可以看到外部代码。右击外部代码显示，这样就可以看到 垃圾wr 做的，双击他，可以跳到一个页面，点击加载就可以。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017915151312.jpg)

这时候可以看到 dotPeek 在反编译，这个时间比较长，需要去做一些你喜欢做的事情，回来就可以发现 dotPeek 反编译好而且你看到 ms 源代码，这时候可以尝试源代码断点，但是不是所有地方都可以断点。

如果你发现无法进入代码，那么尝试安装 Resharper ，如果还是不行，那么需要问一下，是不是使用 UWP ，因为现在我尝试 UWP 还没有成功。

如果还是无法成功，不要来问我，我教了几个小伙伴，有几个是没法进入代码，使用方法都一样，我自己去他电脑弄了，结果我无法进入。

那么接下来就是调试 ms 源代码了，因为已经进入了 Release 的反编译代码，所以通过堆栈调用就进入了源代码，在需要的地方使用断点，当然，不是所有地方可以使用断点。但是进入之后还是可以和原来的调试自己代码一样，看到没有被优化掉的参数的值，可以修改这些值，可以进入其他地方代码设置断点，设置条件，已经使用单步调试跟着代码。

在 win10 下，调试的代码是没有注释的，但是可以对比 dotpeek 的代码来看，一般他里面的代码就是有注释的，反编译的代码和 dotPeek 看到代码有些地方是不同的，但是实际功能是一样的。但是微软源代码使用的框架可能和自己的不一样，看起来代码还是不相同。

最好是自己去下载微软源代码，然后把他放在一个仓库，这样可以看到不同的框架修改的代码。

因为 UWP 编译使用 .netNative ，很多底层都是使用 C++ 写的，所以无法对 UWP 进行反编译

## 下载

[dotPeek32 2018.1.3 CSDN下载](https://download.csdn.net/download/lindexi_gd/10550260 )

[dotPeek64 2018.1.3 CSDN下载](https://download.csdn.net/download/lindexi_gd/10550249 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
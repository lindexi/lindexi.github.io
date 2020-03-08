# win10 uwp 调试软件启动

有一些软件在发布的时候发现软件在启动的时候就退出，无法调试。本文告诉大家如何调试一个 UWP 的启动。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<!-- csdn -->

<!-- 标签：win10,uwp,调试,VisualStudio,VisualStudio调试 -->

<div id="toc"></div>

首先需要更新 VisualStudio 到 2017 以上，如果无法下载 VisualStudio 那么我可以提供种子。

在使用了 Release 发布的 UWP 应用之后，先在自己的设备通过商店安装。

如果发现这时启动就退出，那么需要调试，就是下面的方法。

打开 VisualStudio 注意打开的代码是发布的版本，如果因为软件代码版本不相同出现的端口无法使用，请选择运行源代码不相同。

然后在调试，其他调试里可以看到调试的应用程序包

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201832105635.jpg)

这时选择自己的软件进行调试，注意在第一条语句停止

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20183211213.jpg)

如果不需要从第一条语句，但是断点无法使用，那么点击断点选择源代码不同

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201832111629.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

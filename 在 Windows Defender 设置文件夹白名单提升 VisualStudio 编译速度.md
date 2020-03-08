# 在 Windows Defender 设置文件夹白名单提升 VisualStudio 编译速度

最近在使用 VisualStudio 编译的时候发现编译的速度下降了，原因是编译的时候会创建很多文件，微软自带的杀毒工具会扫描创建的文件，所以降低了编译速度

<!--more-->
<!-- CreateTime:2019/3/15 8:52:58 -->

<!-- csdn -->

在 Windows 安全中心也就是 Windows Defender 里面可以设置文件夹白名单，在这个文件夹里面的文件将不会被扫描。将自己的代码仓库，我会将自己所有的代码都放在一个文件夹，将这个文件夹加入到白名单，可以在编译的时候不会被扫描，这样可以加快编译的速度

打开 Windows Defender 点击病毒和威胁功能

![](http://image.acmx.xyz/lindexi%2F201931584912815)

选择自己的代码仓库

![](http://image.acmx.xyz/lindexi%2F201931584947231)

这里可以选择文件、文件夹和进程，顺便将 VisualStudio 也选择

![](http://image.acmx.xyz/lindexi%2F201931585023863)

有小伙伴认为在编译的时候耗性能最大的是计算，其实现在的程序在编译的时候是 IO 才是最慢的，会有很多的文件读写

通过修改白名单，可以提升一些文件读写速度，我测试了添加了白名单可以有效提升速度。

如果用的不是 WindowsDefender 的小伙伴，也可以在自己的杀毒软件里面添加白名单

如果使用的是国产数字杀毒软件，那么我劝你还是不要做程序员了

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

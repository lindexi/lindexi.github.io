
# Rosyln 如何基于 Microsoft.NET.Sdk 制作源代码包

本文告诉大家如何做源代码包，源代码包的意思是安装的包不是安装dll的方式，而是使用源代码的方式。也就是最后是编译包的源代码而不是添加dll，这个方式是解决想要把项目分小，功能分细，但是不希望项目有很多的 dll，因为如果项目有很多 dll 会让软件打开的时间比较长

<!--more-->


<!-- csdn -->
<!-- 草稿 -->
<div id="toc"></div>

先来告诉大家做这个包的目的，如果是使用分开很多项目，一个项目会创建一个 dll ，在客户端的软件，用户很希望软件点击就打开。但是如果 dll 多了，读取dll文件的时间，加上加载 dll 的时间就会很长，这时用户就需要等待软件启动的时间就比较长。

但是从软件开发的功能，不同的功能应该使用不同的项目，这样才可以尽可能复用代码。为了让项目可以分细，而且减少创建的 dll 库，就需要使用本文的技术。

使用源代码的项目而不是引用 dll 的方法实际上在 Chrome 就是这样做，谷歌的开发就是有很多，大概有几百个项目，但是编译出来的 dll 只有一两个，所以他才有一点击就打开。

## 准备工作

在开始读本文之前，希望大家先了解一些概念，请看[理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://walterlv.github.io/post/understand-the-csproj.html )，这一个文章告诉了大家一些基础，不然在看本文的时候会不知道为什么我需要这样写。

本文使用了修改编译，方法是 [如何编写基于 Microsoft.NET.Sdk 的跨平台的 MSBuild Target - walterlv](https://walterlv.github.io/post/write-msbuild-target.html )，从这个文章可以知道如何修改msbuild的编译




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

# Roslyn 通过 nuget 统一管理信息

在日常的开发中，如果需要发布多个库，多个库之间的版权和作者等信息都是相同的。如果需要每次更改信息都打开项目进行编辑，这个效率是很低的。本文提供一个方式，通过安装一个 nuget 包就可以自动填写信息。

<!--more-->


<!-- csdn -->

<!-- 标签：Roslyn，MSBuild,编译器,nuget,打包 -->

<!-- 草稿 -->

最近我多发布了一些项目，但是发布一个项目需要在 nuget 添加的信息有很多，如 authors 、 owners 、 Company 、 Copyright 而本渣很容易就忘记添加了一些值。

于是我就想，如何才可以让 Roslyn 自动帮我添加一些信息，特别是 Copyright ，因为一年就需要更新时间。

我就想在一个地方更新时间，然后在其他的地方都可以通过安装 nuget 的方式自动更新或通过更新 nuget 的方式更新。

特别是对新人，我就不需要告诉他发布一个 nuget 需要填哪些东西，也不需要担心因为他写错公司，只要他去安装 nuget 就可以。







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
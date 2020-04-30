
# GitHub 如何过滤某个作者的 MR 内容

在 WPF 开源仓库里面有大量的机器人的 MR 但是我想要了解现在 WPF 仓库有多少开发者在贡献代码，此时如何在 GitHub 中过滤某个作者的 MR 内容

<!--more-->


<!-- 发布 -->

在 GitHub 的 Filters 可以通过 `-author` 表示去掉某个作者的内容，如下面链接就是去掉机器人的 WPF 仓库的 MR 有哪些 [https://github.com/dotnet/wpf/pulls?q=is%3Apr+-author%3Aapp%2Fdotnet-maestro](https://github.com/dotnet/wpf/pulls?q=is%3Apr+-author%3Aapp%2Fdotnet-maestro)

在 Filters 的字符串是 `is:pr -author:app/dotnet-maestro` 此时将会去掉机器人的内容

那么如果过滤某个作者的 Issus 呢，将 `is:pr` 替换为 `is:issue ` 就可以





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
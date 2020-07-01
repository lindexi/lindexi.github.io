# 命令行参数使用 json 有哪些坑

本文和大家聊聊在命令行参数里面使用 json 会遇到的坑

<!--more-->
<!-- 发布 -->

## 空格问题

命令行会使用空格分割多个命令，因此 json 里面的格式存在空格时，需要做对应的替换

## 引号问题

这是最坑的问题，按照 json 格式的规则， 他的值使用引号包含。根据 [文档](https://msdn.microsoft.com/en-us/library/system.diagnostics.processstartinfo.arguments(v=vs.110).aspx) 可以了解到，需要使用三个引号作为一个引号的表示

## 换行问题

带格式化的 json 会添加很多换行，而在命令行参数里面传换行就很好玩，请自行干掉

## 字符串长度

一般 json 的长度都很长，而 命令行 参数有长度要求

## 解决方法

尝试将 json 参数写入到本地文件，然后传本地文件路径

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

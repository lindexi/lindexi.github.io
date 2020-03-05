#  C++ 驱动开发 error LNK2019 unresolved external symbol __CheckForDebuggerJustMyCode referenced in function DriverEntry

最近在写一个机器人的时候，发现驱动无法编译通过。本文告诉大家如何解决这个问题。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 VisualStudio 2017 15.8 的版本提供新的功能[ C++ Just My Code Stepping ](https://blogs.msdn.microsoft.com/vcblog/2018/06/29/announcing-jmc-stepping-in-visual-studio/) 会让一些项目编译不通过

```
 error LNK2019: unresolved external symbol __CheckForDebuggerJustMyCode referenced in function DriverEntry
```

这个问题是已知的 https://developercommunity.visualstudio.com/content/problem/302014/dirver-build-debugmode-checkfordebuggerjustmycode.html

解决方法有两个：

1. 在工具选项调试，取消调试我的代码

1. 右击项目属性，在C++的命令行添加 `/JMC-`  点击重新编译

最简单的方法是不要使用 DEBUG 模式编译

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

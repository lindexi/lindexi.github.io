#  C++ 驱动开发 error LNK2019 unresolved external symbol __CheckForDebuggerJustMyCode referenced in function DriverEntry

最近在写一个机器人的时候，发现驱动无法编译通过。本文告诉大家如何解决这个问题。

<!--more-->
<!-- csdn -->

在 VisualStudio 2017 15.8 的版本提供新的功能[ C++ Just My Code Stepping ](https://blogs.msdn.microsoft.com/vcblog/2018/06/29/announcing-jmc-stepping-in-visual-studio/) 会让一些项目编译不通过

```
 error LNK2019: unresolved external symbol __CheckForDebuggerJustMyCode referenced in function DriverEntry
```

这个问题是已知的 https://developercommunity.visualstudio.com/content/problem/302014/dirver-build-debugmode-checkfordebuggerjustmycode.html

解决方法有两个：

1. 在工具选项调试，取消调试我的代码

1. 右击项目属性，在C++的命令行添加 `/JMC-`  点击重新编译

最简单的方法是不要使用 DEBUG 模式编译


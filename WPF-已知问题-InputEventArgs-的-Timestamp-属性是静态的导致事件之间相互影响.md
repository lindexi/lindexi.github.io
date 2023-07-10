
# WPF 已知问题 InputEventArgs 的 Timestamp 属性是静态的导致事件之间相互影响

本文记录一个 WPF 已知的设计问题，当前此问题已经被大佬修复，这个设计问题刚好属于比较边缘的模块，我写了这么多年的代码还没有踩到这个坑一次，也没有听到有谁提到这个坑

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

远古时候，不知道大佬是故意还是失误的在 InputEventArgs 类型里面的 `_timestamp` 字段上加上了 static 关键字，让 static 的 Timestamp 属性依赖一个静态字段，约等于让 Timestamp 属性是静态的。如此将会导致多个 InputEventArgs 之间相互影响

大佬在 GitHub 官方上报告了这个问题，详细请看 [https://github.com/dotnet/wpf/issues/7887](https://github.com/dotnet/wpf/issues/7887)

由于大佬是一个成熟的程序猿了，自己报告的 bug 就自己修了，请看 [https://github.com/dotnet/wpf/pull/7910](https://github.com/dotnet/wpf/pull/7910)

修复的方法十分简单，就是去掉 `_timestamp` 字段上的 static 关键字。于是多个 InputEventArgs 之间就不会相互影响了。但这也破坏了 WPF 的行为，也就不能在 .NET 7 合入了




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
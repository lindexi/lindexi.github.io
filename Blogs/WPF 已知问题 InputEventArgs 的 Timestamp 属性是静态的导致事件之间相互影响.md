本文记录一个 WPF 已知的设计问题，当前此问题已经被大佬修复，这个设计问题刚好属于比较边缘的模块，我写了这么多年的代码还没有踩到这个坑一次，也没有听到有谁提到这个坑

<!--more-->


<!-- CreateTime:2023/7/10 8:56:52 -->
<!-- 发布 -->
<!-- 博客 -->

远古时候，不知道大佬是故意还是失误的在 InputEventArgs 类型里面的 `_timestamp` 字段上加上了 static 关键字，让 static 的 Timestamp 属性依赖一个静态字段，约等于让 Timestamp 属性是静态的。如此将会导致多个 InputEventArgs 之间相互影响

大佬在 GitHub 官方上报告了这个问题，详细请看 [https://github.com/dotnet/wpf/issues/7887](https://github.com/dotnet/wpf/issues/7887)

由于大佬是一个成熟的程序猿了，自己报告的 bug 就自己修了，请看 [https://github.com/dotnet/wpf/pull/7910](https://github.com/dotnet/wpf/pull/7910)

修复的方法十分简单，就是去掉 `_timestamp` 字段上的 static 关键字。于是多个 InputEventArgs 之间就不会相互影响了。但这也破坏了 WPF 的行为，也就不能在 .NET 7 合入了

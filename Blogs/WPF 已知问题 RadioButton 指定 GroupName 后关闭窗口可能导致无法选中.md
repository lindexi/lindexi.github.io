本文记录一个 WPF 已知问题，当 WPF 的 RadioButton 指定 GroupName 且将 IsChecked 状态绑定到 ViewModel 上，将包含以上控件的代码的窗口显示两个，接着关闭其中一个。此时可以看到依然开着的窗口的 RadioButton 控件无法正确在用户界面上点击选中

<!--more-->


<!-- 博客 -->
<!-- 发布 -->

此问题已经报告给 WPF 官方，请看 [https://github.com/dotnet/wpf/issues/2995](https://github.com/dotnet/wpf/issues/2995)

最小复现代码：[https://github.com/walterlv/Walterlv.Issues.RadioButton.GroupName](https://github.com/walterlv/Walterlv.Issues.RadioButton.GroupName)

此问题已被 WPF 官方修复，只需更新 .NET 即可

参考文档： [2020-5-14-WPF的RadioButton指定groupname在window关闭后无法check - huangtengxiao](https://xinyuehtx.github.io/post/WPF%E7%9A%84RadioButton%E6%8C%87%E5%AE%9Agroupname%E5%9C%A8window%E5%85%B3%E9%97%AD%E5%90%8E%E6%97%A0%E6%B3%95check.html )

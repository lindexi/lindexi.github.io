
# WPF 已知问题 RadioButton 指定 GroupName 后关闭窗口可能导致无法选中

本文记录一个 WPF 已知问题，当 WPF 的 RadioButton 指定 GroupName 且将 IsChecked 状态绑定到 ViewModel 上，将包含以上控件的代码的窗口显示两个，接着关闭其中一个。此时可以看到依然开着的窗口的 RadioButton 控件无法正确在用户界面上点击选中

<!--more-->


<!-- CreateTime:2023/10/20 19:29:55 -->

<!-- 博客 -->
<!-- 发布 -->

此问题已经报告给 WPF 官方，请看 [https://github.com/dotnet/wpf/issues/2995](https://github.com/dotnet/wpf/issues/2995)

最小复现代码：[https://github.com/walterlv/Walterlv.Issues.RadioButton.GroupName](https://github.com/walterlv/Walterlv.Issues.RadioButton.GroupName)

此问题已被 WPF 官方修复，只需更新 .NET 即可

参考文档： [2020-5-14-WPF的RadioButton指定groupname在window关闭后无法check - huangtengxiao](https://xinyuehtx.github.io/post/WPF%E7%9A%84RadioButton%E6%8C%87%E5%AE%9Agroupname%E5%9C%A8window%E5%85%B3%E9%97%AD%E5%90%8E%E6%97%A0%E6%B3%95check.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

# WPF 开启Pointer消息存在的坑

本文记录在 WPF 开启 Pointer 消息的坑

<!--more-->


<!-- CreateTime:2019/12/24 14:33:41 -->

<!-- 发布 -->

启用了Pointer之后，调用Textbox.Focus()，起不来屏幕键盘，必须点在它之上才行，触摸在它之上才行

默认 Pointer 消息是使用屏幕绝对坐标而不是窗口坐标

可能存在获取 Stylus 事件时触摸点不准，此时可以通过获取 Touch 代替，详细请看 [WPF will have a touch offset after trun on the WM_Pointer message · Issue #3360 · dotnet/wpf](https://github.com/dotnet/wpf/issues/3360 ) 此问题应该在 [Fix raw stylus data to support per-monitor DPI by rladuca · Pull Request #2891 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2891 ) 修复




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
# WPF 全屏窗口将让 Chrome 97 视频停止播放

无论是使用 WPF 全屏窗口，还是高性能全屏透明窗口，都会在 Chrome 97 以及使用 chromium 对应版本内核的应用的视频停止播放。这是 chromium 的一个优化，因为 chromium 认为，如果有全屏窗口盖在上面，自然此时停止播放视频可以节省资源。然而 chromium 却没有考虑到，有很多会议的应用，虽然是全屏的，但也是透明的，于是此时停止播放视频将是非预期的

<!--more-->
<!-- CreateTime:2022/3/10 11:44:40 -->

<!-- 发布 -->

敲黑板，这次 WPF 是背锅的，这完全是 Chrome 97 自己的优化问题

这是 Chrome 97 的功能，是功能，不是 bug 哦

除了 WPF 的全屏窗口进入前台时，会让 Chrome 97 的应用的视频停止播放。其他任何的 Win32 应用，也能让 Chrome 97 的应用的视频停止播放。因为这是 Chrome 97 在内核里的优化判断，只要有窗口满足 [Windows Native Window Occlusion Detection](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_native_window_occlusion_tracking.md )  文档所描述的条件，将会自动停止视频的播放

此问题已算报告给 chromium 官方，细节请看 [Chrome Occlusion Problem is Back, Help Please? : incremental_games](https://www.reddit.com/r/incremental_games/comments/rxcsjy/chrome_occlusion_problem_is_back_help_please/ )

最佳修复方法为禁用 Chrome 此功能，进入 `chrome://flags` 禁用 `#calculate-native-win-occlusion` 即可

另外一个应该是有坑的方法是更改自己的 WPF 应用，如给应用的窗口设置 Win32 的窗口样式，设置 `WM_Popup` 样式，也能解决此问题，因为绕过了 [Windows Native Window Occlusion Detection](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_native_window_occlusion_tracking.md ) 文档描述的方法。但是加上 `WM_Popup` 样式，一个已知问题是会在 .NET Framework 4.7.1 以下的运行时，敲黑板，不是 SDK 版本，是用户端安装的运行时版本，也许会存在触摸失效问题，详细请看 [dotnet/479874-WPF Touch Stops Working After Prolonged Use of Popups.md at master · Microsoft/dotnet](https://github.com/Microsoft/dotnet/blob/master/releases/net471/KnownIssues/479874-WPF%20Touch%20Stops%20Working%20After%20Prolonged%20Use%20of%20Popups.md )

详细解决方法请参阅 [How to force rendering of video tag content while chrome/electron window is not active for screen sharing? - Stack Overflow](https://stackoverflow.com/a/68685080/6116637)

关于 WPF 高性能全屏透明窗口请看 [WPF 制作高性能的透明背景异形窗口（使用 WindowChrome 而不要使用 AllowsTransparency=True） - walterlv](https://blog.walterlv.com/post/wpf-transparent-window-without-allows-transparency.html ) 和 [WPF 制作支持点击穿透的高性能的透明背景异形窗口](https://blog.lindexi.com/post/WPF-%E5%88%B6%E4%BD%9C%E6%94%AF%E6%8C%81%E7%82%B9%E5%87%BB%E7%A9%BF%E9%80%8F%E7%9A%84%E9%AB%98%E6%80%A7%E8%83%BD%E7%9A%84%E9%80%8F%E6%98%8E%E8%83%8C%E6%99%AF%E5%BC%82%E5%BD%A2%E7%AA%97%E5%8F%A3.html )

此问题由 [lsj](https://blog.sdlsj.net) 找到，我只是写博客的工具

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

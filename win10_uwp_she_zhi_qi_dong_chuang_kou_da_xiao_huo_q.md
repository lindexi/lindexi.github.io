# win10 uwp 设置启动窗口大小  获取窗口大小

本文主要说如何设置我们窗口的启动大小，UWP启动窗口大小。

##设置启动窗口

设置窗口大小

```

            ApplicationView.PreferredLaunchViewSize = new Size(1000, 1000);
            ApplicationView.PreferredLaunchWindowingMode = ApplicationViewWindowingMode.PreferredLaunchViewSize;
```
`ApplicationView.PreferredLaunchWindowingMode `设置UWP窗口全屏

```
ApplicationView.PreferredLaunchWindowingMode = ApplicationViewWindowingMode.FullScreen;
```

设置发现我们的窗口没变小，其实使用下面代码

窗口最小

```
ApplicationView.GetForCurrentView().SetPreferredMinSize(new Size(200, 100));
```

##获得窗口大小

`Window.Current.Bounds.Width`

获取窗口高度

`Window.Current.Bounds.Height`


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


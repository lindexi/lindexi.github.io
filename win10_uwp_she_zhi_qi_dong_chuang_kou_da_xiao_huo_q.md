# win10 uwp 设置启动窗口大小  获取窗口大小


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

窗口最小

```
ApplicationView.GetForCurrentView().SetPreferredMinSize(new Size(200, 100));
```

##获得窗口大小

`Window.Current.Bounds.Width`

获取窗口高度

`Window.Current.Bounds.Height`



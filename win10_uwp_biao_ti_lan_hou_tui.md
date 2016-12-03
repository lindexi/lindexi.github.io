# win10 UWP 标题栏后退
<!--more-->


设置里，标题栏有后退按钮

![这里写图片描述](http://img.blog.csdn.net/20160201125801185)

在win平板，可以有后退键，手机也有

pc可以在标题栏


![这里写图片描述](http://img.blog.csdn.net/20160201130404911)

在`OnLaunched`

```
//最后
     Windows.UI.Core.SystemNavigationManager.GetForCurrentView().BackRequested += BackRequested;       Windows.UI.Core.SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = Windows.UI.Core.AppViewBackButtonVisibility.Visible;

```
BackRequested后退方法
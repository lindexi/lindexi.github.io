# win10 UWP 标题栏后退
<!--more-->

<div id="toc"></div>


设置里，标题栏有后退按钮

![这里写图片描述](http://img.blog.csdn.net/20160201125801185)

在win平板，可以有后退键，手机也有

pc可以在标题栏，打开设置可以看到的那个


![这里写图片描述](http://img.blog.csdn.net/20160201130404911)

如果需要在PC打开，请在`OnLaunched`添加下面代码

```csharp
//最后
     Windows.UI.Core.SystemNavigationManager.GetForCurrentView().BackRequested += BackRequested;     //添加事件  Windows.UI.Core.SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = Windows.UI.Core.AppViewBackButtonVisibility.Visible;

```
BackRequested 后退方法，如何获得参见：[c# 设计模式 责任链.md](c-设计模式-责任链.md)

AppViewBackButtonVisibility 可以设置是否显示后退按钮

其实可以写在任何需要显示后退的地方，注意：如果是异步线程，需要把他放在同步线程

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
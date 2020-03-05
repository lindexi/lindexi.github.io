# win10 UWP 标题栏后退

本文告诉大家如何在 UWP 标题栏添加后退按钮

<!--more-->
<!-- CreateTime:2018/9/14 20:22:08 -->


<div id="toc"></div>


设置里，标题栏有后退按钮，请看下图

<!-- ![这里写图片描述](http://img.blog.csdn.net/20160201125801185) -->

![](http://image.acmx.xyz/lindexi%2F20189142075776)

<!-- ![](image/win10_uwp_biao_ti_lan_hou_tui/win10_uwp_biao_ti_lan_hou_tui0.png) -->

在win平板，可以有后退键，手机也有，但是手机的是物理的，平板的和 PC 的后退是在标题栏做的

如果需要在标题栏显示后退按钮，需要使用下面代码

```csharp
     Windows.UI.Core.SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = Windows.UI.Core.AppViewBackButtonVisibility.Visible;
```

<!-- ![](image/win10_uwp_biao_ti_lan_hou_tui/win10_uwp_biao_ti_lan_hou_tui1.png) -->

![](http://image.acmx.xyz/lindexi%2F20189142092410)

在用户点击标题栏的后退按钮的时候，可以通过下面代码拿到事件

```csharp
	     Windows.UI.Core.SystemNavigationManager.GetForCurrentView().BackRequested += BackRequested; 
```

注意 BackRequested 是自己写的函数。

可以通过 BackRequested 的参数 handle 阻止在手机按下后退键让应用隐藏。

BackRequested 后退方法，如何获得参见：[c# 设计模式 责任链.md](c-设计模式-责任链.md) 注意不要在每个页面的构造都使用添加事件，如果这样子，那么就会出现按一下后退出现你想不到的异常。好的做法是在 Load 添加，Unload 去掉。如果这句代码添加在 ViewModel 需要自己在 ViewModel 关闭去掉添加事件。

如果是手机可以通过引用手机的 sdk 使用下面的代码拿到硬件按钮的返回

```csharp
Windows.Phone.UI.Input.HardwareButtons.BackPressed
```

具体代码请看 [Windows-universal-samples/Samples/BackButton at master · Microsoft/Windows-universal-samples](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/BackButton )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
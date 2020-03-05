# win10 uwp 标题栏

我们应用最上方的是标题栏，对于手机，最上方是状态栏。

我们可以自定义标题栏，和不显示标题栏。

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>
<!-- csdn -->

下面的代码写在 OnLaunched 函数

写的位置是最前。



## 扩展标题栏

参见：http://dailydotnettips.com/2015/10/20/how-to-place-custom-xaml-content-in-the-windows-universal-apps-title-bar/

## 修改颜色

通过下面的方法可以修改颜色 

```csharp
 Windows.UI.Core.SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = Windows.UI.Core.AppViewBackButtonVisibility.Visible;
            ApplicationView.GetForCurrentView().TitleBar.ButtonBackgroundColor = Color.FromArgb(0xFF, 140, 206, 205);
            ApplicationView.GetForCurrentView().TitleBar.ButtonForegroundColor = Color.FromArgb(0xFF, 250, 250, 250);
            ApplicationView.GetForCurrentView().TitleBar.InactiveForegroundColor = Color.FromArgb(0xFF, 250, 250, 250);
```

除了上面的颜色，还有其他的一些颜色可以修改。建议是把颜色写在 xaml 然后后台去拿。

请看 [UWP中实现自定义标题栏](http://www.cnblogs.com/durow/p/4897773.html )

## 透明标题栏 系统状态栏

手机是系统状态栏，如果不设置，会是白条，看起来不好。

我们需要安装sdk才可以。

我的 SDK 是14393，在他之前的使用也一样，在他之后的，如果以后改了，我也不知。

首先判断是不是手机 `ApiInformation.IsTypePresent("Windows.UI.ViewManagement.StatusBar")` 

如果是的话我们可以设置 StatusBar ，上面的字符串最好复制我的，自己打可能打错

我们之前显示的白色，因为背景是空，加上没有前景。

我们可以设置背景的透明，如果设置了0，需要设置前景才看到标题。
		

```csharp
             if (ApiInformation.IsTypePresent("Windows.UI.ViewManagement.StatusBar"))
            {
                var applicationView = ApplicationView.GetForCurrentView();
                applicationView.SetDesiredBoundsMode(ApplicationViewBoundsMode.UseCoreWindow);
                var statusbar = Windows.UI.ViewManagement.StatusBar.GetForCurrentView();
                statusbar.BackgroundColor = Colors.Beige;
                statusbar.BackgroundOpacity = 0.2;
                statusbar.ForegroundColor=Colors.Black;
                
            }  

```

其中 SetDesiredBoundsMode 是设置内容是不是可以占有标题栏

statusbar.BackgroundColor 如果设置透明为0，那么可能看不到系统显示的日期等，我们需要设置前景色。

我们还可以在上面显示Text，设置`statusbar.ProgressIndicator.Text`然后显示，使用函数`ShowAsync`
		

```csharp
statusbar.ProgressIndicator.Text = "test statusbar";  
statusbar.ProgressIndicator.ShowAsync();

```

最后一个显示 Text 可以看微信，他是把消息放在标题栏。

参见：http://uwpbox.com/status-bar-at-the-top-of-the-uwp-statusbar.html

参见：[http://www.cnblogs.com/tcjiaan/p/4783049.html](http://www.cnblogs.com/tcjiaan/p/4783049.html)

![](http://i.wotula.com/wp.png)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
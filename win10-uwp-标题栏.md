# win10 uwp 标题栏

我们应用最上方的是标题栏，对于手机，最上方是状态栏。

我们可以自定义标题栏，和不显示标题栏。

<!--more-->

## 扩展标题栏

参见：http://dailydotnettips.com/2015/10/20/how-to-place-custom-xaml-content-in-the-windows-universal-apps-title-bar/

## 修改颜色

## 透明标题栏 系统状态栏

手机是系统状态栏，如果不设置，会是白条，看起来不好。

我们需要安装sdk才可以。

首先判断是不是手机`ApiInformation.IsTypePresent("Windows.UI.ViewManagement.StatusBar")`

如果是的话我们可以设置StatusBar

		
```C#
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

其中SetDesiredBoundsMode是设置内容是不是可以占有标题栏

statusbar.BackgroundColor如果设置透明为0，那么可能看不到系统显示的日期等，我们需要设置前景色。

我们还可以在上面显示Text，设置`statusbar.ProgressIndicator.Text`然后显示，使用函数`ShowAsync`
		
```C#
statusbar.ProgressIndicator.Text = "test statusbar";  
statusbar.ProgressIndicator.ShowAsync();

```

参见：http://uwpbox.com/status-bar-at-the-top-of-the-uwp-statusbar.html

参见：http://www.cnblogs.com/tcjiaan/p/4783049.html
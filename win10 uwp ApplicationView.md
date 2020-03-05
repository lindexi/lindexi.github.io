# win10 uwp ApplicationView

本文和大家介绍一个重要的类，他可以用来设置窗口，如设置启动大小，设置是否允许截图，是否进入全屏，所有和窗口有关的，都可以在他这里设置。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


<div id="toc"></div>

可以使用简单获取`ApplicationView applicationView = ApplicationView.GetForCurrentView();` ，注意，他是不能构造创建

先从属性开始

第一个属性是 AdjacentToLeftDisplayEdge ，判断是不是靠近屏幕左边，在和屏幕只有20像素之内就是，但是其他包括窗口在屏幕外就不是。和他差不多的有属性 AdjacentToRightDisplayEdge 判断是否靠近右边缘。

这个属性和获得窗口变化一起使用。

如何获得窗口大小变化？使用 CoreWindow.GetForCurrentThread().SizeChanged 可以获得窗口大小变化，可以获得是否靠近屏幕左边。

第二是 `DesiredBoundsMode` 感觉没什么用，指示框架用于排列窗口内容边界值

 - `FullScreenSystemOverlayMode` 全屏响应手势的模式，包括手势可调用系统覆盖（标题 任务栏）， 边缘手势可调用临时 UI，而此UIElement反过来可调用对于该边缘的系统覆盖

 - 获取窗口 ID，使用 applicationView.Id ，对于UWP可以使用多个窗口，因为需要区分是哪个窗口，所以就可以使用这个。

 - 如何判断是否进入全屏？使用 applicationView.IsFullScreenMode 。判断是否全屏的功能，和他相同的 applicationView.IsFullScreen 也可以，但是这个不推荐

 - 如何获得窗口处于锁屏？ applicationView.IsOnLockScreen 我自己尝试锁屏但是依旧无法获得，如果知道这个是什么，请告诉我。

 - 如何设置可以不截图？有些应用需要有秘密，不可以被截图，可以使用`applicationView.IsScreenCaptureEnabled` 如果设置 false，所有应用难以对程序截图。如果对于显示密码或其他的页面，是要求打开这个设置，也就是在输入密码时，是需要设置不可截图，如果设置了，那么截图是黑色，无法获得。在不显示密码时，可以设置为 true，这时可以截图。

 首先做一个简单的按钮，在点击他就设置为false，不可截图。在点击前，使用 Q 截图，发现可以

 ![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017429512.jpg)

 点击之后，截图看到的

 ![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201742962.jpg)

 如果设置了，可以看到，但是不可以获取，可以对信息安全要求高的程序进行使用，但是设置了性能可能没有之前好。

 关于不可截图，代码：https://code.msdn.microsoft.com/windowsapps/Disable-screen-capture-00efe630

 这个对于应用内截图是不是还可以使用，我还没去试，如果有大神做了，可以的话，请告诉。

 - 如何获取窗口是横向显示？applicationView.Orientation 可以获取窗口是宽度大于高度，也就是横向，还是相反。

 不是使用屏幕反向，是软件的宽度等，是不是看起来是横向

 - PreferredLaunchViewSize 这个属性是设置窗口大小使用，在设置 PreferredLaunchWindowingMode 为 PreferredLaunchViewSize 可以使用，使用方法：


```csharp
           ApplicationView.PreferredLaunchWindowingMode = ApplicationViewWindowingMode.PreferredLaunchViewSize;
            ApplicationView.PreferredLaunchViewSize = new Size(100, 100);
            如果设置无效，那么在最前设置             applicationView.SetPreferredMinSize(new Size(100,100)); 

```

 PreferredLaunchWindowingMode ms建议使用 auto

 - 如果对于多窗口程序，可以设置 TerminateAppOnFinalViewClose 在关闭所有窗口关闭程序

 - 如何设置窗口标题？使用 applicationView.Title = "标题" 设置会在标题显示 "标题-程序"

 - 设置标题栏颜色 applicationView.TitleBar 可以获得标题栏，可以通过设置属性自定义标题栏

   - BackgroundColor ![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20174292037.jpg)

   - ButtonBackgroundColor ![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20174292343.jpg)

 - 获取窗口可见区 applicationView.VisibleBounds ，通过 applicationView.VisibleBoundsChanged 可以知道隐藏或显示标题栏会改变窗口可见大小的方法

 - 退出全屏  applicationView.ExitFullScreenMode(); 只有在桌面才可以使用

 - 进入全屏 applicationView.TryEnterFullScreenMode()

 - 修改窗口大小 applicationView.TryResizeView() 只有桌面才可以使用

 - 全屏显示标题栏 applicationView.ShowStandardSystemOverlays() 




参见：https://docs.microsoft.com/en-us/uwp/api/windows.ui.viewmanagement.applicationview

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
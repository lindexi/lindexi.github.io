# win10 uwp 设置启动窗口大小  获取窗口大小

本文主要说如何设置我们窗口的启动大小，UWP启动窗口大小。
<!--more-->

##设置启动窗口

设置窗口大小

```

            ApplicationView.PreferredLaunchViewSize = new Size(1000, 1000);
            ApplicationView.PreferredLaunchWindowingMode = ApplicationViewWindowingMode.PreferredLaunchViewSize;
```
`ApplicationView.PreferredLaunchWindowingMode `设置UWP窗口全屏

在手机没有用，手机就全屏，其他没用

如果设置过屏幕大小导致了每次开启窗口都变小，那么可以简单使用（下面代码没有测试）
```
ApplicationView.PreferredLaunchWindowingMode = ApplicationViewWindowingMode.PreferredLaunchViewSize;

ApplicationView.PreferredLaunchWindowingMode =
ApplicationViewWindowingMode.Auto;
```
和这个问题相似的还有，UWP的标题栏问题，我们通过设置了ExtendViewIntoTitleBar=true，导致了没有标题栏，但是如果我们之后设置了false，程序关闭后发现并没有用，简单的方法

```
            var windows = CoreApplication.GetCurrentView().TitleBar;
            windows.ExtendViewIntoTitleBar = false;
            windows.ExtendViewIntoTitleBar = true;
```

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

但是如果我们需要判断我们的窗口大小变化的话，一个简单的方法，使用动态适应

```
       <VisualStateManager.VisualStateGroups >
            <VisualStateGroup CurrentStateChanged="{x:Bind View.NarrowVisual}">
                <VisualState>
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowWidth="720"/>
                    </VisualState.StateTriggers>
                    <VisualState.Setters >
                        <!--<Setter Target="Img.Visibility" Value="Collapsed"></Setter>-->
                    </VisualState.Setters>
                </VisualState>
                <VisualState>
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowHeight="200">

                        </AdaptiveTrigger>

                    </VisualState.StateTriggers>
                    <VisualState.Setters >

                    </VisualState.Setters>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
```

在后台绑定变化，我写在View的变窄。

然后在View写我们拿到窗口大小

```
        public void NarrowVisual(object sender, VisualStateChangedEventArgs e)
        {
            //Window.Current.Bounds.Width
            //Window.Current.Bounds.Height
        }

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


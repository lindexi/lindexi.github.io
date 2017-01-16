# win10 UWP 全屏

win10 可以全屏软件或窗口，窗口有一般、最小化、最大化。我们有新的API设置我们软件是全屏，是窗口。我们可以使用`ApplicationView`让我们软件全屏，取消。

<!--more-->

下面是一个简单的例子，判断我们软件是不是全屏，如果是，就不全屏，代码在一个ToggleButton

```csharp
ApplicationView view = ApplicationView.GetForCurrentView();

bool isInFullScreenMode = view.IsFullScreenMode;

if (isInFullScreenMode)  
{
    view.ExitFullScreenMode();
}
else  
{
    view.TryEnterFullScreenMode();
}
```
`IsFullScreenMode`为true，现在应用全屏

`ExitFullScreenMode`退出全屏

`TryEnterFullScreenMode`进入全屏，进入全屏成功true

如果窗口改变需要知道，可以注册`Window.Current.SizeChanged`

```csharp
public class IsFullScreenModeTrigger : StateTriggerBase  
{
    public IsFullScreenModeTrigger()
    {
        ApplicationView view = ApplicationView.GetForCurrentView();

        SetActive(view.IsFullScreenMode);

        Window.Current.SizeChanged += CurrentWindow_SizeChanged;
    }

    private void CurrentWindow_SizeChanged(object sender, Windows.UI.Core.WindowSizeChangedEventArgs e)
    {
        ApplicationView view = ApplicationView.GetForCurrentView();

        SetActive(view.IsFullScreenMode);
    }
}
```

```xml
<Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">  
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup>
            <VisualState x:Name="InFullScreen">
                <VisualState.StateTriggers>
                    <local:IsFullScreenModeTrigger />
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="IsFullScreenText.Text" Value="In full screen mode" />
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>
    <StackPanel HorizontalAlignment="Center">
        <Button Click="Button_Click" 
                Content="Toggle full screen"></Button>
        <TextBlock x:Name="IsFullScreenText" 
                    FontSize="72" 
                    TextWrapping="Wrap" 
                    Text="Not in full screen mode"/>
    </StackPanel>
</Grid>  
```
在我们应用变为全屏，textblock就会`In full screen mode`

我们可以设置`PreferredLaunchWindowingMode`，在我们应用打开

```csharp
ApplicationView.PreferredLaunchWindowingMode = ApplicationViewWindowingMode.FullScreen;
```
ApplicationViewWindowingMode可以`Auto` ,`PreferredLaunchViewSize`设置窗口和`ApplicationView.PreferredLaunchViewSize`，如果没有设置`ApplicationView.PreferredLaunchViewSize`会使用上次关闭窗口， `FullScreen`

win10有很简单的API可以应用全屏，在电脑，我们经常用窗口，手机经常使用全屏。

http://igrali.com/2015/06/21/full-screen-mode-in-windows-10-universal-apps/

## C++ 全屏

<script src="https://gist.github.com/gyakoo/cfef3ca0403d26a082afc8c055240082.js"></script>

https://gist.github.com/gyakoo/cfef3ca0403d26a082afc8c055240082

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



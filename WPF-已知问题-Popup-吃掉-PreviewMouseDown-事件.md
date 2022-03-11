
# WPF 已知问题 Popup 吃掉 PreviewMouseDown 事件

在 WPF 中，使用 Popup 也许会看到 PreviewMouseDown 事件被吃掉

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

因为 PreviewMouseDown 是 RoutingStrategy.Direct 路由事件，不能在多个视觉树使用，在设置 Popup 点击界面 StaysOpen="False" 的逻辑就在下面代码

```csharp
private void OnPreviewMouseButton(MouseButtonEventArgs e)
{
    // We should only react to mouse buttons if we are in an auto close mode (where we have capture)
    if (_cacheValid[(int)CacheBits.CaptureEngaged] && !StaysOpen)
    {
        Debug.Assert( Mouse.Captured == _popupRoot.Value, "_cacheValid[(int)CacheBits.CaptureEngaged] == true but Mouse.Captured != _popupRoot");

        // If we got a mouse press/release and the mouse isn't on the popup (popup root), dismiss.
        // When captured to subtree, source will be the captured element for events outside the popup.
        if (_popupRoot.Value != null && e.OriginalSource == _popupRoot.Value)
        {
            // When we have capture we will get all mouse button up/down messages.
            // We should close if the press was outside.  The MouseButtonEventArgs don't tell whether we get this
            // message because we have capture or if it was legit, so we have to do a hit test.
            if (_popupRoot.Value.InputHitTest(e.GetPosition(_popupRoot.Value)) == null)
            {
                // The hit test didn't find any element; that means the click happened outside the popup.
                SetCurrentValueInternal(IsOpenProperty, BooleanBoxes.FalseBox);
            }
        }
    }
}
```

如果写一个 CheckBox 放在界面上，运行代码可以看到可以被打勾但是没有事件

```xml
<Grid>
    <StackPanel>
        <Button Margin="10,10,10,10" Content="Open Popup" Click="OpenPopup_OnClick"></Button>
        <CheckBox PreviewMouseDown="UIElement_OnPreviewMouseDown"></CheckBox>
    </StackPanel>
    <Popup x:Name="Popup" StaysOpen="False">
        <Grid Width="100" Height="100" Background="White">
            <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" Text="Popup"></TextBlock>
        </Grid>
    </Popup>
</Grid>
```

在 `UIElement_OnPreviewMouseDown` 添加输出内容，代码如下，可以看到，没有符合预期输出

```csharp
private void UIElement_OnPreviewMouseDown(object sender, MouseButtonEventArgs e)
{
    Debug.WriteLine("PreviewMouseDown");
}
 
private void OpenPopup_OnClick(object sender, RoutedEventArgs e)
{
    Popup.PlacementTarget = (UIElement) sender;
    Popup.Placement = PlacementMode.Mouse;
    Popup.IsOpen = true;
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/46e813ad18655df1653e1fb9de6c238f91171443/NayfarwehelaFebeejochar) 欢迎访问

此问题已报告 WPF 官方，请看 [Known issus: Popup with “StaysOpen=false” steals PreviewMouseDown event · Issue #2166 · dotnet/wpf](https://github.com/dotnet/wpf/issues/2166 )

更多请看 [dotnet 读 WPF 源代码 Popup 的 StaysOpen 为 false 将会吃掉其他窗口的首次激活](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81-Popup-%E7%9A%84-StaysOpen-%E4%B8%BA-false-%E5%B0%86%E4%BC%9A%E5%90%83%E6%8E%89%E5%85%B6%E4%BB%96%E7%AA%97%E5%8F%A3%E7%9A%84%E9%A6%96%E6%AC%A1%E6%BF%80%E6%B4%BB.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
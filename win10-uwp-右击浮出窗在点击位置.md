# win10 uwp 右击浮出窗在点击位置

如果需要让 Flyout 显示在指定的位置，那么请看本文。

本文主要让 MenuFlyout 出现在我们右击位置。

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

我们一般使用的 MenuFlyout 写在前台，写在 Button 里面，但是可能我们的 MenuFlyout 显示的位置和我们想要的不一样。

也可能是为了使用  ToggleButton ，而他里面没有 FLyOut ，需要使用基类来写，所以这时就需要在其他控件写的 Flyout 放在指定控件的显示。如果需要获得控件的坐标，请看 [win10 uwp 获得元素绝对坐标](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E8%8E%B7%E5%BE%97%E5%85%83%E7%B4%A0%E7%BB%9D%E5%AF%B9%E5%9D%90%E6%A0%87/)。本文使用的方法是在 后台代码使用 MenuFlyout ，然后在后台进行显示，需要知道的是，这个方法不能直接在前台完成。

通过使用后台写ShowAt的方法，我们可以通过 e.GetPosition 获得鼠标点击位置，需要对函数传入相对的元素，这个元素一般可以用我们点击使用的元素，也可以使用我们的最外层Grid，这样我们就可以获得了鼠标位置，也就可以显示我们的 MenuFlyout 在点击位置。

我们建一个ListView，然后绑定后台，在我们ListView要右击显示我们的浮出，要求我们的浮出在我们点击位置。

MenuFlyout可以在后台写，当然写在前台也可以，但是让他显示在指定位置的就必须在后台代码写。

我们下面的代码写在后台，我们可以选择 Placement  显示在我们元素的位置，但这不是我们鼠标点击的位置，要显示我们鼠标点击的位置，其实也很简单。我们可以从`e.GetPosition(sender as UIElement)`获得鼠标位置，把这个给`MenuFlyout`我们的浮出显示在我们鼠标点击位置。

    <ListView ItemsSource="{x:Bind View.Str}">
            <ListView.ItemContainerStyle>
                <Style TargetType="ListViewItem">
                    <Setter Property="HorizontalContentAlignment"
                                        Value="Stretch" />
                    <Setter Property="VerticalContentAlignment" Value="Center"></Setter>
                </Style>
            </ListView.ItemContainerStyle>
            
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid Background="#FFda2a5c" RightTapped="GridColection_OnRightTapped">
                        <TextBlock Text="{Binding}"></TextBlock>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>

后台写获取鼠标位置、把浮出窗放在鼠标位置

       private void GridColection_OnRightTapped(object sender, RightTappedRoutedEventArgs e)
        {
            MenuFlyout myFlyout = new MenuFlyout();
            MenuFlyoutItem firstItem = new MenuFlyoutItem { Text = "OneIt" };
            MenuFlyoutItem secondItem = new MenuFlyoutItem { Text = "TwoIt" };
            myFlyout.Items.Add(firstItem);
            myFlyout.Items.Add(secondItem);

            //if you only want to show in left or buttom 
            //myFlyout.Placement = FlyoutPlacementMode.Left;
            
            FrameworkElement senderElement = sender as FrameworkElement;
            //the code can show the flyout in your mouse click 
            myFlyout.ShowAt(sender as UIElement, e.GetPosition(sender as UIElement)); 如果需要显示在某个控件，就拿到控件的坐标
        }
 
 于是上面的代码就可以做出下面的这张图，点击的时候显示浮出，在点击的位置。

![](https://ooo.0o0.ooo/2016/10/01/57ef223c62d80.gif)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




#win10 uwp 右击浮出窗在点击位置

【】

我们建一个ListView，然后绑定后台，在我们ListView

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

后台写

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
            myFlyout.ShowAt(sender as UIElement, e.GetPosition(sender as UIElement));
        }
 
![](https://ooo.0o0.ooo/2016/10/01/57ef223c62d80.gif)


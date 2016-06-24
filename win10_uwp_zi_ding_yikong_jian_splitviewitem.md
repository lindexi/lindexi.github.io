# win10 uwp 自定义控件 SplitViewItem


我们使用汉堡菜单，经常需要一个
![这里写图片描述](http://img.blog.csdn.net/20160624111821645)
需要一个图标和一个文字

我开始写

```
                            <ListViewItem.Content>
                                <StackPanel Orientation="Horizontal">
                                    <TextBlock Margin="10,10,10,10" FontFamily="Segoe MDL2 Assets"
                                         Text="&#xE77B;"></TextBlock>
                                    <TextBlock Margin="10,10,10,10" Text="登录"></TextBlock>
                                </StackPanel>
                            </ListViewItem.Content>
```

因为需要写3个，我觉得复制不好，因为我还有很多软件，如果每个都这样，那么在TextBlock使用
![这里写图片描述](http://img.blog.csdn.net/20160624112019381)
很多都是一样的

自己创建控件，右击添加控件

在控件

```
    <Grid>
        <StackPanel Orientation="Horizontal">
            <TextBlock Margin="10,10,10,10" FontFamily="Segoe MDL2 Assets"
                       Text="{x:Bind IconString}"></TextBlock>
            <TextBlock Margin="10,10,10,10" Text="{x:Bind Text}"></TextBlock>
        </StackPanel>
    </Grid>
```

然后在`SplitViewItem.xaml.cs`

属性IconString，Text

```
        public static readonly DependencyProperty IconStringProperty = DependencyProperty.Register(
            "IconString", typeof(string), typeof(SplitViewItem), new PropertyMetadata(default(string)));

        public string IconString
        {
            set
            {
                SetValue(IconStringProperty, value);
            }
            get
            {
                return (string) GetValue(IconStringProperty);
            }
        }

        public static readonly DependencyProperty TextProperty = DependencyProperty.Register(
            "Text", typeof(string), typeof(SplitViewItem), new PropertyMetadata(default(string)));

        public string Text
        {
            set
            {
                SetValue(TextProperty, value);
            }
            get
            {
                return (string) GetValue(TextProperty);
            }
        }
```

我把SplitViewItem扔View文件夹，在使用

`    xmlns:view="using:EncryptionSyncFolder.View"`

本来需要很长的，现在修改

```
                        <ListViewItem>
                            <ListViewItem.Content>
                               <Grid>
                                    <view:SplitViewItem IconString="&#xE713;" Text="设置"></view:SplitViewItem>
                               </Grid>
                            </ListViewItem.Content>
                        </ListViewItem>
```


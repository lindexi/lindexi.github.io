# win10 uwp 自定义控件 SplitViewItem


本文主要是因为汉堡菜单里面列出的菜单很多重复的图标和文字，我把它作为控件，因为是随便写，可能存在错误，如果发现了，请和我说或关掉浏览器，请不要发不良言论。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

我们使用汉堡菜单，经常需要一个
![这里写图片描述](http://img.blog.csdn.net/20160624111821645)
需要一个图标和一个文字

我开始写一个TextBlock做图标，一个写文字

```xml
                            <ListViewItem.Content>
                                <StackPanel Orientation="Horizontal">
                                    <TextBlock Margin="10,10,10,10" FontFamily="Segoe MDL2 Assets"
                                         Text="&#xE77B;"></TextBlock>
                                    <TextBlock Margin="10,10,10,10" Text="登录"></TextBlock>
                                </StackPanel>
                            </ListViewItem.Content>
```

因为需要写3个地方是不一样，一个是文字、一个图标还有一次复制，我觉得复制不好，因为我还有很多软件，如果每个都这样，那么在TextBlock使用
![这里写图片描述](http://img.blog.csdn.net/20160624112019381)
很多都是一样的

一个方法是自己创建控件，我们右击View文件夹添加控件

在控件写两个TextBlock，一个做图标，一个写文字

```xml
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

```csharp
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

我把SplitViewItem扔View文件夹，在命名空间使用`EncryptionSyncFolder.View`

`    xmlns:view="using:EncryptionSyncFolder.View"`

本来需要很长的代码，现在修改成为一点点，其实就是导入我的自定义控件，首先在上面的代码是把view用作我的控件所在文件夹，反人类的Segoe MDL2 Assets 可以在http://modernicons.io/segoe-mdl2/cheatsheet/，找到你要的图标

```xml
                        <ListViewItem>
                            <ListViewItem.Content>
                               <Grid>
                                    <view:SplitViewItem IconString="&#xE713;" Text="设置"></view:SplitViewItem>
                               </Grid>
                            </ListViewItem.Content>
                        </ListViewItem>
```

## SplitView 从右划出

修改PanelPlacement。

多谢[Script](http://i.hexun.com/bzlbsd/default.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


# win10 uwp 弹起键盘不隐藏界面元素

本文主要讲，在我们使用手机输入的时候，会因为手机的虚拟键盘隐藏了一些界面的元素。我们有一个简单的方法让虚拟键盘不隐藏界面元素。

<!--more-->
<!-- CreateTime:2020/3/5 9:26:17 -->


<div id="toc"></div>

我们需要的界面元素是在显示了虚拟键盘后的空间能全部显示，如果不能的话，还是会被隐藏的。

我们可以选择在弹起虚拟键盘时，修改界面布局。

首先是应用sdk，手机的。

通过手机虚拟键盘显示和关闭`InputPane.GetForCurrentView().Showing`和`InputPane.GetForCurrentView().Hiding`可以后台修改界面。

做一个测试界面，很简单

```xml
<Grid>
            <Grid.RowDefinitions>
                <RowDefinition></RowDefinition>
                <!--显示虚拟键盘 他的高度会为虚拟键盘高度-->
                <RowDefinition x:Name="HightKeyboard" Height="Auto"></RowDefinition>
            </Grid.RowDefinitions>
            <!--这里才是原本的界面-->
            <!--原本的使用VerticalAlignment="Center"-->
            <Grid>
                <!--这里写界面-->
                <StackPanel Orientation="Vertical"
                          VerticalAlignment="Center">
                    <TextBlock Text="我是界面"
                         HorizontalAlignment="Center"></TextBlock>
                    <TextBox Margin="10,10,10,10" Header="输入"></TextBox>
                    <TextBlock Margin="10,100,10,10" Text="手机点击输入 会隐藏我"
                               HorizontalAlignment="Center"></TextBlock>
                    <TextBlock></TextBlock>
                </StackPanel>
            </Grid>
        </Grid>
```

简单的界面：

![](http://image.acmx.xyz/ae470125-e6d5-452b-8b4a-0c54bf2e5d3220161130214749.jpg)

写这个代码写在Main.xaml.cs不在ViewModel，界面变化是View要做的

代码写MainPage()

```csharp
            InputPane.GetForCurrentView().Showing += (s, e) =>
            {
                HightKeyboard.Height=new GridLength(e.OccludedRect.Height);
            };

            InputPane.GetForCurrentView().Hiding += (s, e) =>
            {
                HightKeyboard.Height=new GridLength(1);
            };
```

我的私密密码本在输入就使用隐藏，开始是界面有图，当弹起键盘，显示主要的

![](http://image.acmx.xyz/0a7537fb-9ef0-49f8-b934-6cb779e8754bwp_ss_20161204_00012016124201938.jpg)

![](http://image.acmx.xyz/0a7537fb-9ef0-49f8-b934-6cb779e8754bwp_ss_20161204_00022016124202039.jpg)

http://www.cnblogs.com/manupstairs/p/5738387.html

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


#win10 uwp 车表盘 径向规

【】

车表盘就是有刻度的圆盘加上针，这个控件可以直观让用户知道当前的速度或其他

看名字不知道是什么，我就放一张图

![这里写图片描述](http://img.blog.csdn.net/20160903165703064) 

![这里写图片描述](http://img.blog.csdn.net/20160903173217169) 

使用很简单，在Nuget，Radial Gauge

![这里写图片描述](http://img.blog.csdn.net/20160903165948549) 


要使用大神做的，简单，在使用我们需要在Nuget下载，然后在引用` xmlns:controls="using:WinRTXamlToolkit.Controls"`

我们需要知道每个值是什么

![这里写图片描述](http://img.blog.csdn.net/20160903173511567) 

tick:最上面表盘，我们可以修改颜色TickBrush，我们颜色可以用SolidColorBrush、ImageBrush、LinearGradientBrush

我们对每个值都可以修改，可以是颜色，图片。

trail:我们可以修改颜色，如果需要修改大小，我们修改下面的scaleTick

scaleTick:可以修改颜色，大小，ScaleWidth就是大小，ScaleTickBrush颜色

scale:还没有使用的，可以设置颜色

Minimum：我们可以修改和进度条那最大值，最小值

Unit：下面写的字，我写了博客

needle：指针，英文我就不翻译

```
            <controls:Gauge
                x:Name="Gauge"
                Value="10"
                Unit="http://blog.csdn.net/lindexi_gd"
                ScaleTickBrush="Gainsboro"
                TickBrush="Black"
                ScaleWidth="50"
                Margin="1,50,1,100" Width="600">
                <controls:Gauge.TrailBrush>
                    <SolidColorBrush Color="Red"></SolidColorBrush>
                </controls:Gauge.TrailBrush>
                <controls:Gauge.ScaleBrush>
                    <SolidColorBrush Color="Cornsilk"></SolidColorBrush>
                </controls:Gauge.ScaleBrush>
                <controls:Gauge.NeedleBrush>
                    <LinearGradientBrush EndPoint="1,0">
                        <GradientStop Color="Transparent" />
                        <GradientStop Color="Goldenrod"
                          Offset="0.5" />
                        <GradientStop Color="Transparent"
                          Offset="1" />
                    </LinearGradientBrush>
                </controls:Gauge.NeedleBrush>
            </controls:Gauge>
```

如果感兴趣可以去https://github.com/xyzzer/WinRTXamlToolkit




[http://blogs.u2u.be/diederik/post/2015/12/14/The-Radial-Gauge-goes-UWP.aspx](http://blogs.u2u.be/diederik/post/2015/12/14/The-Radial-Gauge-goes-UWP.aspx)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




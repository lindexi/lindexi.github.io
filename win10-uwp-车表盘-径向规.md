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


[http://blogs.u2u.be/diederik/post/2015/12/14/The-Radial-Gauge-goes-UWP.aspx](http://blogs.u2u.be/diederik/post/2015/12/14/The-Radial-Gauge-goes-UWP.aspx)




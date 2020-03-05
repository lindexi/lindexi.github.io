# WPF 解决 ViewBox  不显示线的问题

ViewBox 是一个好用的东西，但是在他缩小的时候，可能有一些线无法显示。

现在公司项目就是做一个类似 ppt 的软件，所以需要使用缩略图，而对于矩形形状，在缩略图，经常看不到线。

因为 ViewBox 和 visualBrush 都使用 邻近算法 所以  ViewBox 和 visualBrush  都存在丢失线的问题。

本文提供一个算法，解决 单线条在WPF不显示问题。1像素线段在WPF不显示问题。ViewBox 缩小失去线段问题。

<!--more-->
<!-- CreateTime:2018/11/21 9:37:53 -->

<!-- csdn -->
<div id="toc"></div>

我发现这个问题，于是在 堆栈网提问：[https://stackoverflow.com/q/44495238/6116637](https://stackoverflow.com/q/44495238/6116637)，最后在[walterlv](https://github.com/walterlv) 的帮助下，找到解决方法。

先来说下问题：

如果使用 ViewBox 缩小一个矩形，如果线段只有 1 像素，那么容易就丢失。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017613101115.jpg)

请看上图，左边就是一个矩形，右边是使用 ViewBox 做出来的缩小图形。可以看到存在线条不显示，但是在移动矩形过程中，有些线就显示了，于是看起来图形在闪烁，这个设计不好。

当然为了显示矩形，我需要使用 VisualBrush 。为了说明 ViewBox 问题，我用了两个方法，一个就是使用 一个ViewBox 里面放矩形。一个就是使用 ViusalBrush 显示矩形。得到结果差不多， ViewBox 和 visualBrush 都会丢失线段。


```csharp
         <Border x:Name="SlideBorder" Margin="10,10,10,100" BorderThickness="1" BorderBrush="White">
                <Border.Background>
                    <VisualBrush  Visual="{Binding ElementName=Rectangle}" Stretch="Uniform"  />
                </Border.Background>
            </Border>
```


```csharp
            <Viewbox  >
            <Grid x:Name="Rectangle" Background="#FFFFFFFF">
                <Rectangle Margin="10,50,10,10" Stroke="#FF565656"
                           StrokeThickness="1"
                           UseLayoutRounding="True">

                </Rectangle>
                </Grid>
                
            </Viewbox>
```


但是大家都知道，ps 缩小图片，不会容易就出现线段不显示，于是能否使用和 ps 相似的方法？

答案，是的。于是使用的技术有：控件截图、改变图片大小

通过控件截图得到控件的图片，然后通过改变图片大小方式，不会让线段不显示。

## wpf 截图

可以使用下面代码截图，width 是图片像素宽度，height是高度


```csharp
             var bitmap = new RenderTargetBitmap(width, height, 96.0 dpi 就是96, 96.0, PixelFormats.Pbgra32);
            bitmap.Render(控件);
```

如果dpi不是96，那么请使用其他值


通过上面方法就可以截图，然后需要修改图片大小。

## 修改图片大小

修改图片大小，可以使用`TransformedBitmap`

如果需要把图片修改为大小为 size ，请使用下面代码，这个代码的效率很高。


```csharp
    new TransformedBitmap(bitmap, new ScaleTransform(size.Width / 图片宽度, size.Height / 图片高度))
```

这样可以返回一个 BitmapImage ，于是就得到从输入一个控件到输出一个图片

通过上面的方法，可以使用和 VisualBrush 的方法，把控件转为图片，但是效率没有 visualBrush 那么高。不过在 `1280*720P` 的控件效率大概比 VisualBrush 时间大概多不到 50 毫秒。当然我的配置比较高也有关，TransformedBitmap 的代码是在 GPU 计算的，而截图是在 UI 线程，所以需要注意一下。

总的代码就是：


```csharp
        public static BitmapSource ToBitmapSource(Visual visual, Size size)
        {
            var bounds = VisualTreeHelper.GetDescendantBounds(visual);
            var width = (int) Math.Round(bounds.Width);
            var height = (int) Math.Round(bounds.Height);
            var bitmap = new RenderTargetBitmap(width, height, 96.0, 96.0, PixelFormats.Pbgra32);
            bitmap.Render(visual);
            return new TransformedBitmap(bitmap, new ScaleTransform(size.Width / width, size.Height / height));
        }
```

输入你需要转换的控件，输入转换后的大小，得到一个图片

于是大概就是 VisualBrush 的功能。

于是使用上面的代码，尝试缩小，可以看到不会丢失线

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20176131139.jpg)

缺点：无法获得用户的输入，得到是图片，只能用于显示

大法的缩略图，是在用户输入完成在做新的图片，尝试移动一个图片，在移动中，缩略图是不显示的。

呆磨：http://download.csdn.net/detail/lindexi_gd/9868941

参见：[How to fix VisualBrush lost line?](https://stackoverflow.com/q/44495238/6116637)

[how to avoid a single pixel line disappear in wpf?](https://stackoverflow.com/q/29552339/6116637)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
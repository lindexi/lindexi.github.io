# win10 uwp 获得元素绝对坐标

有时候需要获得一个元素，相对窗口的坐标，在修改他的位置可以使用。

那么 UWP 如何获得元素坐标？

我提供了一个方法，可以获得元素的坐标。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>

首先需要获得元素，如果没有获得元素，那么如何得到他的坐标？

假如 xaml 是这样，而我需要获得 MainTextBlock 相对窗口的坐标


```csharp
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <TextBlock x:Name="MainTextBlock" Margin="10,10,10,10" Text="Hello" />
    </Grid>
```

那么获得元素绝对坐标可以这样写，绝对坐标的意思就是元素相对窗口的坐标。


```csharp
         public MainPage()
        {
            this.InitializeComponent();
            var t = MainTextBlock.TransformToVisual(Window.Current.Content);
            Point screenCoords = t.TransformPoint(new Point(0, 0));
        }
```

上面代码就可以获得元素坐标，坐标相对于窗口

那么如何获得他相对其他元素的坐标？

假如需要获得元素相对他的上坐标，这时可以看下面代码


```csharp
            var t = MainTextBlock.TransformToVisual((UIElement)MainTextBlock.Parent);
            Point screenCoords = t.TransformPoint(new Point(0, 0));
```

于是可以看到 `TransformToVisual` 传入的是哪个元素，就是获得相对于这个元素的坐标。

获得元素的坐标有什么用？可以用在如 Flyout的定位，如果使用了  ToggleButton ，他没有自己 Flyout ，所以就需要在其他地方定义一个 Flyout 然后通过获得控件位置显示出来。如何指定 Flyout 的位置参见 [win10 uwp 右击浮出窗在点击位置 ](http://blog.csdn.net/lindexi_gd/article/details/52724410)

所以就可以让浮出窗在需要显示的按钮上显示，下面的图片是我偷一个大神的，他就是使用这个方法做出来。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017627974.jpg)


参见：http://stackoverflow.com/questions/12387449/how-to-get-the-absolute-position-of-an-element/12388558#12388558

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
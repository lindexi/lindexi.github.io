# win10 uwp 绘图  Line 控件使用

本文主要讲一个在绘图中，我们会有一个基础的控件，Line。控件的基本使用和他能做出的我们很多时候需要的界面。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>

虽然是一个简单控件，但是可以做出很诡异的很好看的UI。

首先，我们要知道，Line就是画直线。当然我们画他一般是在Canvas中。

我们先在一个页面新建一下，写下必要的代码。

```xml
       <Grid Margin="10,10,10,10">
            <Canvas>
                
            </Canvas>
        </Grid>

```

首先我们需要确定直线坐标，用X1X2，Y1Y2来获得两个点，也就是直线。


```xml
 <Line X1="0" X2="200" Y1="10" Y2="200"></Line>

```

有了直线，可以在设计看到一条线，但是运行是没有看到的，因为我们没有给线大小。

![](http://image.acmx.xyz/3d1cee81-4688-4db3-80d6-14ea8c9ce64c20161125145232.jpg)

Stroke就是线段设置颜色或用其他渐变填充，StrokeThickness就是线段的大小

我们写上`Stroke="Black" StrokeThickness="5"`

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515218.jpg)

如果我们把大小写为50，那么就会看到很不好看

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515413.jpg)

于是我们想要裁剪他，裁剪可以用Clip

我们用矩形来裁剪他。

矩形需要四个点。

可以看到线段裁剪了

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515727.jpg)

被一个大小为150的矩形裁剪了

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515828.jpg)

接着就是ms-uap 写的 http://www.cnblogs.com/ms-uap/p/4641419.html

我们给他一个StrokeDashArray，这个值就是很多数，奇数的数就是显示宽度，偶数的就是不显示的宽度，读到最后的数就循环到最前。我们设置一个0.1一个0.2就会显示0.1的宽度，然后空0.2再显示0.1

```xml
               <Line X1="0" X2="200" Y1="10" Y2="200"
                      Stroke="Black" StrokeThickness="50"
                      StrokeDashArray="0.1 0.2">
                    <Line.Clip>
                        <RectangleGeometry Rect="0 0 150 150"></RectangleGeometry>
                    </Line.Clip>
                </Line>
```

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac9520161125151239.jpg)

需要知道，0.1的宽度是`0.1*StrokeThickness`的宽度

我们把宽度大一些

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac9520161125151552.jpg)

然后加个线条，就可以做出ms-uap写的图

我们还可以设置线段的开始图形，要知道，我们线段画比较大，那么看起来就是矩形，StrokeStartLineCap就可以让线段看起来好看

```csharp
StrokeStartLineCap="Round"
```

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac9520161125151939.jpg)

```csharp
StrokeStartLineCap="Triangle"
```

![](http://image.acmx.xyz/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515214.jpg)

线段两头尖的画法就是在设置后面的`StrokeEndLineCap`

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

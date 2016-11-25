----------
Title: win10 uwp 绘图  Line 控件使用
----------

在绘图中，我们会有一个基础的控件，Line。

虽然是一个简单控件，但是可以做出很诡异的很好看的UI。

首先，我们要知道，Line就是画直线。当然我们画他一般是在Canvas中。

我们先在一个页面新建一下，写下必要的代码。

```
       <Grid Margin="10,10,10,10">
            <Canvas>
                
            </Canvas>
        </Grid>

```

首先我们需要确定直线坐标，用X1X2，Y1Y2来获得两个点，也就是直线。


```
 <Line X1="0" X2="200" Y1="10" Y2="200"></Line>

```

有了直线，可以在设计看到一条线，但是运行是没有看到的，因为我们没有给线大小。

![](http://7xqpl8.com1.z0.glb.clouddn.com/3d1cee81-4688-4db3-80d6-14ea8c9ce64c20161125145232.jpg)

Stroke就是设置颜色或用其他渐变填充，StrokeThickness就是线段的大小

我们写上`Stroke="Black" StrokeThickness="5"`

![](http://7xqpl8.com1.z0.glb.clouddn.com/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515218.jpg)

如果我们把大小写为50，那么就会看到很不好看

![](http://7xqpl8.com1.z0.glb.clouddn.com/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515413.jpg)

于是我们想要裁剪他，裁剪可以用Clip

我们用矩形来裁剪他。

矩形需要四个点。

可以看到线段裁剪了

![](http://7xqpl8.com1.z0.glb.clouddn.com/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515727.jpg)

被一个大小为150的矩形裁剪了

![](http://7xqpl8.com1.z0.glb.clouddn.com/a6d555d5-766d-4ca9-83d6-51270fdaac952016112515828.jpg)

接着就是ms-uap 写的 http://www.cnblogs.com/ms-uap/p/4641419.html

我们给他一个StrokeDashArray，这个值就是很多数，奇数的数就是显示宽度，偶数的就是不显示的宽度，读到最后的数就循环到最前。我们设置一个0.1一个0.2就会显示0.1的宽度，然后空0.2再显示0.1

```
               <Line X1="0" X2="200" Y1="10" Y2="200"
                      Stroke="Black" StrokeThickness="50"
                      StrokeDashArray="0.1 0.2">
                    <Line.Clip>
                        <RectangleGeometry Rect="0 0 150 150"></RectangleGeometry>
                    </Line.Clip>
                </Line>
```



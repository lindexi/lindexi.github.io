# win10 UWP 用Path画图

本文将使用 Path 画一个聊天气泡。

<!-- csdn -->
<!--more-->

<div id="toc"></div>

内容是看到 大神写的 [WPF绘制简单常用的Path](http://www.cnblogs.com/tsliwei/p/5609035.html)，想到 UWP 画图是不是也一样，于是做的一个抄袭的 Path

直接使用图片

![](http://images2015.cnblogs.com/blog/918258/201606/918258-20160622230949985-738039663.png)

我们写上所有点。写在折线，在UWP，还是存在和 WPF 做法有些修改，却没有修改什么。


```csharp
            <Path Stroke="Black" StrokeThickness="2" Margin="10,10,10,10">
              <Path.Data>
                  <PathGeometry>
                        <PathGeometry.Figures>
                            <PathFigure>
                                <PolyLineSegment Points="0,0 100,0 100,90 55,90 50,100 45,90 0,90 0,0"></PolyLineSegment>
                            </PathFigure>
                        </PathGeometry.Figures>
                    </PathGeometry>
              </Path.Data>
          </Path>
```

其实，可以不加`PathGeometry.Figures` ，看起来就和之前代码一样

![这里写图片描述](http://img.blog.csdn.net/20170122150822837?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGluZGV4aV9nZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面就使用 ArcSegment 看起来不是尖角

![这里写图片描述](http://img.blog.csdn.net/20170122151858822?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGluZGV4aV9nZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


```csharp
              <Path Stroke="Black" StrokeThickness="2" Margin="200,10,10,10">
                <Path.Data>
                    <PathGeometry>
                        <PathGeometry.Figures>
                            <PathFigure StartPoint="0,5">
                                <LineSegment Point="0,85"></LineSegment>
                                <ArcSegment Point="5,90" Size="5,5"></ArcSegment>
                                <LineSegment Point="45,90"></LineSegment>
                                <LineSegment Point="50 100"></LineSegment>
                                <LineSegment Point="55,90"></LineSegment>
                                <LineSegment Point="95,90"></LineSegment>
                                <ArcSegment Point="100,85" Size="5,5"></ArcSegment>
                                <LineSegment Point="100,5"></LineSegment>
                                <ArcSegment Point="95,0" Size="5,5" ></ArcSegment>
                                <LineSegment Point="5,0"></LineSegment>
                                <ArcSegment Point="0,5" Size="5,5"></ArcSegment>
                            </PathFigure>
                        </PathGeometry.Figures>
                    </PathGeometry>
                </Path.Data>
            </Path>

```

如果我们把第一个图，边框变大，可以看到没有合，这样觉得不好

![这里写图片描述](http://img.blog.csdn.net/20170122152402395?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGluZGV4aV9nZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以设置 `PathFigure IsClosed="True"` 让最后一个点合在开始，这样就是闭合，不会出现没有合

我们来说下 ArcSegment 






http://www.cnblogs.com/tsliwei/p/5609035.html

http://www.cnblogs.com/xpvincent/p/3830108.html
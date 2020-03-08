# WPF DrawingVisual

本文：如何自定义控件用 DrawingVisual 画图

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

本文不会讲 DrawingVisual 是什么，只会告诉简单方法画图。

为何需要学这个？如果需要画出图形，对性能有要求，或者需要了解WPF如何画图，就需要知道这个。

先创建最简单使用，就是显示文字或显示点。

我觉得显示文字简单，于是开始写代码，先不要去想做什么，代码需要一个控件和一个画出文字的类。

首先新建一个控件，他是可以让 DrawingVisual 显示。


```csharp
    public class MyVisualHost : FrameworkElement
```

这是很基础一个类，几乎没有什么功能。

于是新建一个  FrameworkElement  需要添加 一些方法，这是默认的，只需要自动创建就好。

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017323102952.jpg)

这个类不是主要的，他是让DrawingVisual显示，在构造函数写下面的代码

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017323103719.jpg)


这就是可以让 他可以显示。为何这样可以，参见：http://blog.csdn.net/changtianshuiyue/article/details/26981797

主要的类StrokeVisual，其实很简单，他可以在上面的类显示文字


```csharp
        public class StrokeVisual : DrawingVisual

```
来看下他的方法 

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732310406.jpg)

这样就可以画出文字。

需要在xaml添加下面代码，就可以显示出来


```csharp
            <local:MyVisualHost></local:MyVisualHost>

```


![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732310419.jpg)

为什么这样就可以画出？

那么如何做一个鼠标点下就画点的软件？

调用 RenderOpen 就可以打开一个 DrawingContext ，他提供很多方法，在他上面使用就可以画出，不过画出来看不到。需要添加到FrameworkElement才可以。

那么如何做出下图的程序？

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017%25E5%25B9%25B43%25E6%259C%258823%25E6%2597%25A5%2520112045.gif)

首先对代码做修改，在 Windows 的MouseMove 调用 StrokeVisual 的 Add 方法和 画出来

需要获得鼠标的位置，获得方法很简单，在 MouseMove 函数写下面的代码，其中 e 就是参数


```csharp
    p=e.GetPosition(this);
```

传入 StrokeVisual 调用他的 Draw 可以看到他画出来了


```csharp
            _s.Add(new StylusPoint(p.X, p.Y));
            _s.Draw();
```

那么需要看下添加的函数如何写，下面代码就是整个 StrokeVisual 的代码。

```csharp
        public StrokeVisual()
        {
            Stroke = new Stroke(new StylusPointCollection(new Point[] { new Point(10, 10), }), new DrawingAttributes()
            {
            });


        }

        public void Add(StylusPoint point)
        {
            Stroke.StylusPoints.Add(point);
        }

        private Stroke Stroke;
```
那么如何从 Stroke 画出？

可以使用 Stroke 传入 dc 就可以画出来。


```csharp
            using (var dc = RenderOpen())
            {
                Stroke.Draw(dc);
            }
```

代码很简单，建议自己去写，我就不把代码给你。如果自己无法写，需要代码，那么[联系](mailto:lindexi_gd@163.com)我发代码给你。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

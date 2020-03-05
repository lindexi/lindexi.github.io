# WPF 等距布局

本文告诉大家如何使用 WPF 的自定义布局做等距布局。

<!--more-->
<!-- CreateTime:2019/10/31 9:00:02 -->

<!-- 标签：wpf，布局 -->

实际做的效果很简单，因为在开发我容易就用到了等距的控件。等距控件就是在指定的宽度下，平均把控件放在水平的地方，这样相等于 StackPanel 的水平，但是没有做水平压缩。在这个控件，无论在水平放多少个控件，都会在相同的高度把他们放下。如果里面的控件的宽度不相同，那么这些控件拿到的可以使用的宽度都是相同。

请看下面的图片

![](http://image.acmx.xyz/cc09f7ce-2059-5d67-bc56-036f04c13efa%2F2018218111032.jpg)

上面图片是存在 6 个按钮的，所有的按钮使用的宽度都是一样

现在修改为 3 个按钮，可以看到说有按钮都是相同

![](http://image.acmx.xyz/cc09f7ce-2059-5d67-bc56-036f04c13efa%2F2018218111254.jpg)

现在加两个宽度很小的文本，可以看到文本可以的宽度和按钮一样

![](http://image.acmx.xyz/cc09f7ce-2059-5d67-bc56-036f04c13efa%2F2018218111430.jpg)

下面让我告诉大家这个控件是如何做。

在 WPF 做自己的面板可以继承Panel ，可以重写两个方法，第一个方法是 MeasureOverride ，重写这个方法可以告诉上一级控件，这个控件需要多大的空间。第二个方法是 ArrangeOverride 告诉元素可以怎么放。

下面创建一个类 KbiseczvTom 这是等距控件。

首先重写MeasureOverride，因为需要的一般只是做水平等距，所以就需要拿到元素的宽度和高度，把所有的宽度合起来作为这个控件需要的最小宽度，然后拿到所有控件的最大高度作为这个控件的需要高度。虽然从 MeasureOverride 返回了大小，但是实际上的上一级控件是不是最后给这么大的，还是不知道的。

```
        protected override Size MeasureOverride(Size availableSize)
        {
            var size = new Size();
            foreach (var temp in Children.Cast<UIElement>())
            {
                temp.Measure(new Size(double.PositiveInfinity, double.PositiveInfinity));
                size = new Size(size.Width + temp.DesiredSize.Width, Math.Max(size.Height, temp.DesiredSize.Height));
            }
            return size;
        }

```    

在调用 Measure 传入无穷是因为这是需要让元素告诉控件需要的最大大小。

然后就是重写 ArrangeOverride ，传入的参数就是上一级控件给这个控件的大小，返回值就是实际需要的大小。现在可以使用元素的 Arrange 通过这个可以把元素给元素的大小，左上角。但是元素是不是就听话，实际上还是不知道的。首先拿到元素数，把拿到的宽度除元素得到一个元素可以使用宽度，然后把每个元素按照顺序给左上角，宽度。


```
        protected override Size ArrangeOverride(Size availableSize)
        {
            var size = availableSize;

            var width = size.Width / Children.Count;

            for (int i = 0; i < Children.Count; i++)
            {
                var temp = Children[i];
                temp.Arrange(new Rect(new Point(i*width,0),new Size(width,size.Height)));
            }

            return size;
        }

```    

现在打开界面写下面代码试试

```
       <local:KbiseczvTom HorizontalAlignment="Stretch" Height="100">
            <Button Margin="10,10,10,10" Content="点击"></Button>
            <Button Margin="10,10,10,10" Content="点击"></Button>
            <Button Margin="10,10,10,10" Content="点击"></Button>

            <TextBlock VerticalAlignment="Center" Text="文本"></TextBlock>
            <TextBlock VerticalAlignment="Center" Text="文本"></TextBlock>
        </local:KbiseczvTom>
```    

所有代码：
 

```
    public class KbiseczvTom : Panel
    {
        protected override Size MeasureOverride(Size availableSize)
        {
            var size = new Size();
            foreach (var temp in Children.Cast<UIElement>())
            {
                temp.Measure(new Size(double.PositiveInfinity, double.PositiveInfinity));
                size = new Size(size.Width + temp.DesiredSize.Width, Math.Max(size.Height, temp.DesiredSize.Height));
            }
            return size;
        }

        protected override Size ArrangeOverride(Size availableSize)
        {
            var size = availableSize;

            var width = size.Width / Children.Count;

            for (int i = 0; i < Children.Count; i++)
            {
                var temp = Children[i];
                temp.Arrange(new Rect(new Point(i*width,0),new Size(width,size.Height)));
            }

            return size;
        }
    }

```    

源代码：[WPF 等距布局-CSDN下载](http://download.csdn.net/download/lindexi_gd/10254406 )

因为现在的 csdn 是需要下载积分，所以我会把一些资源放到 RetroShare ，这是一个很好的分享工具，如何使用请看[使用 RetroShare 分享资源 ](https://lindexi.oschina.io/lindexi/post/%E4%BD%BF%E7%94%A8-RetroShare-%E5%88%86%E4%BA%AB%E8%B5%84%E6%BA%90.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

# win10 uwp 拖动控件

我们会使用控件拖动，可以让我们做出好看的动画，那么我们如何移动控件，我将会告诉大家多个方法。其中第一个是最差的，最后的才是我希望大神你去用。

<!--more-->
<!-- CreateTime:2020/3/5 9:26:17 -->


<div id="toc"></div>

## Margin 移动

我们可以使用Margin移动，但这是wr说不要这样做。

> We can move the control by Margin,but using this method is not recommended.

我们可以在xaml写一个Button，然后就使用左键获取鼠标，这个可以去看 win10 uwp 获取按钮鼠标左键按下

http://lindexi.oschina.io/lindexi/post/win10-uwp-%E8%8E%B7%E5%8F%96%E6%8C%89%E9%92%AE%E9%BC%A0%E6%A0%87%E5%B7%A6%E9%94%AE%E6%8C%89%E4%B8%8B/

于是在Button_OnPointerMoved，我们获取移动的xy

		

```csharp
PointerPoint point = e.GetCurrentPoint(btn);

```

这样`point.Position.X`就是移动的左边

我们可以通过`x += point.Position.X - btn.ActualWidth / 2.0;`

这是因为`btn.ActualWidth / 2.0`不用的话会是控件的左上角。

我们把它给Margin

		

```csharp
        private void Button_OnPointerMoved(object sender, PointerRoutedEventArgs e)
        {
            Button btn=sender as Button;
            if (btn == null)
            {
                return;
            }
            e.Handled = true;

            PointerPoint point = e.GetCurrentPoint(btn);

            if (point.Properties.IsLeftButtonPressed)
            {
                double x = (double)btn.GetValue(Canvas.LeftProperty);
                double y = (double)btn.GetValue(Canvas.TopProperty);
                x += point.Position.X - btn.ActualWidth / 2.0;
                y += point.Position.Y - btn.ActualHeight / 2.0;
                btn.Margin=new Thickness(x,y,0,0);
            }
        }

```

## Canvas 拖动控件

我们需要把控件放在Canvas，然后使用Margin一样的

我们需要设置附件属性，`btn.SetValue(Canvas.LeftProperty, x)`就是设置`Canvas.Left`
		

```csharp
        private void Button_OnPointerMoved(object sender, PointerRoutedEventArgs e)
        {
            Button btn=sender as Button;
            if (btn == null)
            {
                return;
            }
            e.Handled = true;

            PointerPoint point = e.GetCurrentPoint(btn);

            if (point.Properties.IsLeftButtonPressed)
            {
                double x = (double)btn.GetValue(Canvas.LeftProperty);
                double y = (double)btn.GetValue(Canvas.TopProperty);
                x += point.Position.X - btn.ActualWidth / 2.0;
                y += point.Position.Y - btn.ActualHeight / 2.0;
                btn.SetValue(Canvas.LeftProperty, x);
                btn.SetValue(Canvas.TopProperty, y);
            }
        }

```

## Manipulation 拖动控件

我们可以使用手势，这个需要在控件设置`ManipulationMode="All"`，使用`ManipulationDelta`

```csharp
        private void Button_OnManipulationDelta(object sender, ManipulationDeltaRoutedEventArgs e)
        {
            Button btn = sender as Button;
            if (btn == null)
            {
                return;
            }

            if (dragTranslation == null)
            {
                dragTranslation = new TranslateTransform();
            }

            btn.RenderTransform = dragTranslation;

            dragTranslation.X += e.Delta.Translation.X;
            dragTranslation.Y += e.Delta.Translation.Y;
        }

```

做好之后，我们发现实在奇怪

![](http://image.acmx.xyz/%E6%8B%96%E5%8A%A8%E6%8E%A7%E4%BB%B62016%E5%B9%B412%E6%9C%8818%E6%97%A5151534.gif)


```csharp
大神，请用力划。

大神：我的控件哪去？

控件：谁叫你那么用力

Canvas：我的左边可以长度无限。

……

```

好在[OneWindows](https://leoldev.wordpress.com/2016/12/18/uwp-manipulationdelta？蛤？/)的帮助

参见：http://www.cnblogs.com/cjw1115/p/5323339.html

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
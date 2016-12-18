# win10 uwp 拖动控件

我们会使用控件拖动，可以让我们做出好看的动画，那么我们如何移动控件，我将会告诉大家多个方法。其中第一个是最差的，最后的才是我希望大神你去用。

<!--more-->

## Margin 移动

我们可以使用Margin移动，但这是wr说不要这样做。

我们可以在xaml写一个Button，然后就使用左键获取鼠标，这个可以去看 win10 uwp 获取按钮鼠标左键按下

http://lindexi.oschina.io/lindexi/post/win10-uwp-%E8%8E%B7%E5%8F%96%E6%8C%89%E9%92%AE%E9%BC%A0%E6%A0%87%E5%B7%A6%E9%94%AE%E6%8C%89%E4%B8%8B/

于是在Button_OnPointerMoved，我们获取移动的xy

		
```
PointerPoint point = e.GetCurrentPoint(btn);

```

这样`point.Position.X`就是移动的左边

我们可以通过`x += point.Position.X - btn.ActualWidth / 2.0;`

这是因为`btn.ActualWidth / 2.0`不用的话会是控件的左上角。

我们把它给Margin

		
```
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
		
```
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

		
```
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
![](http://7xqpl8.com1.z0.glb.clouddn.com/%E6%8B%96%E5%8A%A8%E6%8E%A7%E4%BB%B62016%E5%B9%B412%E6%9C%8818%E6%97%A5151534.gif)


参见：http://www.cnblogs.com/cjw1115/p/5323339.html
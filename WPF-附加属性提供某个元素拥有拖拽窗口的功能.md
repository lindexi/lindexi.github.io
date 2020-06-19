
# WPF 附加属性提供某个元素拥有拖拽窗口的功能

我需要窗口内的某个元素拥有拖动整个窗口的功能，也就是这个元素在拖动的时候是拖动整个窗口。而且我还开出一个有趣的方法，这个作为窗口的拖拽的元素如果是用户在元素上拖动，那么将会拖动窗口，如果用户是点击，将会触发点击事件

<!--more-->


<!-- CreateTime:6/16/2020 7:44:54 PM -->

<!-- 发布 -->

附加属性可以给某个元素附加有趣的功能，本文的功能需要拖动元素的时候实际上是拖动窗口，第二个是元素是支持点击的

拖动窗口使用的是窗口的 DragMove 方法

元素支持点击用的是 [WPF 给任意控件通过按下移动抬起封装点击事件](https://blog.lindexi.com/post/WPF-%E7%BB%99%E4%BB%BB%E6%84%8F%E6%8E%A7%E4%BB%B6%E9%80%9A%E8%BF%87%E6%8C%89%E4%B8%8B%E7%A7%BB%E5%8A%A8%E6%8A%AC%E8%B5%B7%E5%B0%81%E8%A3%85%E7%82%B9%E5%87%BB%E4%BA%8B%E4%BB%B6.html ) 方法

因此本文需要引入 [WPF 给任意控件通过按下移动抬起封装点击事件](https://blog.lindexi.com/post/WPF-%E7%BB%99%E4%BB%BB%E6%84%8F%E6%8E%A7%E4%BB%B6%E9%80%9A%E8%BF%87%E6%8C%89%E4%B8%8B%E7%A7%BB%E5%8A%A8%E6%8A%AC%E8%B5%B7%E5%B0%81%E8%A3%85%E7%82%B9%E5%87%BB%E4%BA%8B%E4%BB%B6.html ) 的 InputHelper 类作为辅助

用法如下，写一个简单的界面，放一个元素作为拖动的元素

```xml
<Border x:Name="DraggingElement" Width="100" Height="100" Background="Gray">
 

    <TextBlock x:Name="TextBlock" Margin="10,10,10,10" HorizontalAlignment="Center"></TextBlock>
</Border>
```

上面代码的 DraggingElement 就是用来拖动窗口的元素

接下来在元素放一个 WindowDraggingExtension.DragWindow 附加属性

```xml
<Border x:Name="DraggingElement" Width="100" Height="100" Background="Gray">

   <framework:WindowDraggingExtension.DragWindow>
        <framework:WindowDraggingExtension />
    </framework:WindowDraggingExtension.DragWindow> 

    <TextBlock x:Name="TextBlock" Margin="10,10,10,10" HorizontalAlignment="Center"></TextBlock>
</Border>
```

注意 `framework:` 是我的命名空间，请按照自己的代码修改为你的命名空间

此时尝试运行代码，拖动一下 DraggingElement 这个元素，可以看到拖动的是窗口。这个方法支持触摸拖动

这个附加属性能做到的功能类似 QQ 宠物，可以拖动，可以点击提示更多内容

本文用到的这个附加属性代码如下

```csharp
    /// <summary>
    /// 窗口拖拽的附加方法
    /// </summary>
    public class WindowDraggingExtension
    {
        /// <summary>
        /// 表示元素作为附加某个窗口提供拖拽的功能
        /// </summary>
        public static readonly DependencyProperty DragWindowProperty = DependencyProperty.RegisterAttached(
            "DragWindow", typeof(WindowDraggingExtension), typeof(WindowDraggingExtension),
            new PropertyMetadata(default(WindowDraggingExtension),
                OnDragWindowPropertyChanged));

        /// <summary>
        /// 附加的拖动的窗口，提供此属性仅仅是为了提升性能，可以不设置。如不设置将使用 Window.GetWindow 方法获取当前元素所在窗口
        /// </summary>
        public Window TargetWindow { set; get; }

        /// <summary>
        /// 拖动的元素实际是被点击时触发
        /// </summary>
        public event EventHandler DraggingElementClicked;

        /// <summary>
        /// 拖动时触发
        /// </summary>
        public event EventHandler Dragging;

        /// <summary>
        /// 设置元素作为窗口的拖拽元素
        /// </summary>
        /// <param name="element"></param>
        /// <param name="value"></param>
        public static void SetDragWindow(DependencyObject element, WindowDraggingExtension value)
        {
            element.SetValue(DragWindowProperty, value);
        }

        /// <summary>
        /// 获取元素作为窗口拖拽属性
        /// </summary>
        /// <param name="element"></param>
        /// <returns></returns>
        public static WindowDraggingExtension GetDragWindow(DependencyObject element)
        {
            return (WindowDraggingExtension) element.GetValue(DragWindowProperty);
        }

        private static void OnDragWindowPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            // 仅有设置，不会存在多次设置，也没有反过来
            if (e.NewValue is WindowDraggingExtension windowDragging && d is UIElement element)
            {
                InputHelper.AttachMouseDownMoveUpToClick(element,
                    delegate { windowDragging.OnDraggingElementClicked(); }, delegate
                    {
                        windowDragging.OnDragging();

                        if (Mouse.LeftButton == MouseButtonState.Pressed)
                        {
                            var targetWindow = windowDragging.TargetWindow
                                               ?? Window.GetWindow(element);

                            targetWindow?.DragMove();
                        }
                    });
            }
        }

        private void OnDraggingElementClicked()
        {
            DraggingElementClicked?.Invoke(this, EventArgs.Empty);
        }

        private void OnDragging()
        {
            Dragging?.Invoke(this, EventArgs.Empty);
        }
    }
```

上面代码 InputHelper 需要从 [WPF 给任意控件通过按下移动抬起封装点击事件](https://blog.lindexi.com/post/WPF-%E7%BB%99%E4%BB%BB%E6%84%8F%E6%8E%A7%E4%BB%B6%E9%80%9A%E8%BF%87%E6%8C%89%E4%B8%8B%E7%A7%BB%E5%8A%A8%E6%8A%AC%E8%B5%B7%E5%B0%81%E8%A3%85%E7%82%B9%E5%87%BB%E4%BA%8B%E4%BB%B6.html ) 复制

通过阅读上面代码，可以看到还有两个可以设置的属性，一个是 TargetWindow 属性，一个是元素被点击的事件

设置 TargetWindow 属性主要是为了提升一点性能，通过 TargetWindow 获取窗口，而不需要通过 Window.GetWindow 方法获取当前元素所在窗口，使用方法如下

```xml
<Window x:Name="CurrentWindow" 忽略元素>

  <Border x:Name="DraggingElement" Width="100" Height="100" Background="Gray">

     <framework:WindowDraggingExtension.DragWindow>
          <framework:WindowDraggingExtension TargetWindow="{x:Reference CurrentWindow}" DraggingElementClicked="WindowDraggingExtension_OnDraggingElementClicked"/>
      </framework:WindowDraggingExtension.DragWindow> 

      <TextBlock x:Name="TextBlock" Margin="10,10,10,10" HorizontalAlignment="Center"></TextBlock>
  </Border>
</Window>
```

上面代码还使用方法拿到元素点击的事件，后台代码如下

```csharp
        private void WindowDraggingExtension_OnDraggingElementClicked(object sender, EventArgs e)
        {
            TextBlock.Text = "林德熙是逗比";
        }
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
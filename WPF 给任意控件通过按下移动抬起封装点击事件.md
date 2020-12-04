# WPF 给任意控件通过按下移动抬起封装点击事件

其实点击这个事件是可以通过按下移动和抬起三个事件封装出来的，本文提供给大家一个辅助的方法，方便给任意的控件附加点击事件

<!--more-->
<!-- CreateTime:6/15/2020 2:55:27 PM -->



在开始前需要了解一些本文点击的定义，本文点击的定义就是在按下到抬起过程中，不会出现长距离的移动，也就是从点击到抬起的点都是在一定范围内的。同时可以设置一定的时间范围，超过一定时间就是长按了，而不是点击了

根据这个特点咱开始来进行一个简单的封装，在封装之前先告诉大家封装之后的使用方法，下面代码的 uiElement 是一个 UIElement 控件

```csharp
            InputHelper.AttachMouseDownMoveUpToClick(uiElement, UIElement_OnClicked);

        private void UIElement_OnClicked(object sender, EventArgs e)
        {
            
        }
```

实现 InputHelper 请看下面代码

```csharp
    /// <summary>
    /// 输入层的帮助类
    /// </summary>
    public static class InputHelper
    {
        /// <summary>
        /// 将 MouseDown MouseMove MouseUp 封装为点击事件
        /// </summary>
        /// <param name="element">要被附加的元素</param>
        /// <param name="clickEventHandler">点击的事件</param>
        /// <param name="dragStarted">因为拖动而结束点击时触发</param>
        public static void AttachMouseDownMoveUpToClick(UIElement element, EventHandler clickEventHandler,
            EventHandler dragStarted = null)
        {
            var inputInfo = GetOrCreateInputInfo(element);
            inputInfo.ClickEventHandler += clickEventHandler;

            inputInfo.DragStarted += dragStarted;

            element.MouseDown -= Element_MouseDown;
            element.MouseDown += Element_MouseDown;
            element.MouseMove -= Element_MouseMove;
            element.MouseMove += Element_MouseMove;
            element.MouseUp -= Element_MouseUp;
            element.MouseUp += Element_MouseUp;
            element.LostMouseCapture -= Element_LostMouseCapture;
            element.LostMouseCapture += Element_LostMouseCapture;
        }

        /// <summary>
        /// 去掉对 <paramref name="element"/> 的点击时间的监听
        /// </summary>
        /// <param name="element"></param>
        /// <param name="clickEventHandler">点击的事件</param>
        /// <param name="dragStarted">因为拖动而结束点击时触发的事件</param>
        public static void DetachMouseDownMoveUpToClick(UIElement element, EventHandler clickEventHandler,
            EventHandler dragStarted = null)
        {
            var inputInfo = GetInputInfo(element);
            if (inputInfo == null)
            {
                return;
            }

            inputInfo.ClickEventHandler -= clickEventHandler;
            inputInfo.DragStarted -= dragStarted;

            if (inputInfo.IsEmpty())
            {
                element.ClearValue(InputInfoProperty);
                element.MouseDown -= Element_MouseDown;
                element.MouseMove -= Element_MouseMove;
                element.MouseUp -= Element_MouseUp;
                element.LostMouseCapture -= Element_LostMouseCapture;
            }
        }

        private static void Element_LostMouseCapture(object sender, MouseEventArgs e)
        {
            var element = (UIElement) sender;
            GetInputInfo(element)?.LostCapture();
        }

        private static void Element_MouseUp(object sender, MouseButtonEventArgs e)
        {
            var element = (UIElement) sender;

            GetInputInfo(element)?.Up(e.GetPosition(element));
        }

        private static void Element_MouseMove(object sender, MouseEventArgs e)
        {
            var element = (UIElement) sender;

            GetInputInfo(element)?.Move(e.GetPosition(element));
        }

        private static void Element_MouseDown(object sender, MouseButtonEventArgs e)
        {
            var element = (UIElement) sender;

            GetInputInfo(element)?.Down(e.GetPosition(element));
        }

        private static readonly DependencyProperty InputInfoProperty = DependencyProperty.RegisterAttached(
            "InputInfo", typeof(InputInfo), typeof(InputHelper), new PropertyMetadata(default(InputInfo)));

        private static InputInfo GetOrCreateInputInfo(UIElement element)
        {
            var inputInfo = GetInputInfo(element);
            if (inputInfo == null)
            {
                inputInfo = new InputInfo();
                SetInputInfo(element, inputInfo);
            }

            return inputInfo;
        }

        private static void SetInputInfo(DependencyObject element, InputInfo value)
        {
            element.SetValue(InputInfoProperty, value);
        }

        private static InputInfo GetInputInfo(DependencyObject element)
        {
            return (InputInfo) element.GetValue(InputInfoProperty);
        }

        private class InputInfo
        {
            public void Down(Point position)
            {
                _downedPosition = position;
                _downedTime = DateTime.Now;
                _isClick = true;
            }

            public void Move(Point position)
            {
                if (!_isClick) return;

                if ((position - _downedPosition).LengthSquared > ToleranceSquared)
                {
                    _isClick = false;
                    DragStarted?.Invoke(null, EventArgs.Empty);
                }
            }

            public void Up(Point position)
            {
                _isClick = _isClick
                           && (position - _downedPosition).LengthSquared <= ToleranceSquared
                           && DateTime.Now - _downedTime < ClickDuringTime;

                if (!_isClick) return;

                ClickEventHandler?.Invoke(null, EventArgs.Empty);

                _isClick = false;
            }

            public void LostCapture()
            {
                _isClick = false;
            }

            public double ToleranceSquared { set; get; } = 0.01;
            public TimeSpan ClickDuringTime { set; get; } = TimeSpan.MaxValue;

            public event EventHandler ClickEventHandler;
            public event EventHandler DragStarted;

            public bool IsEmpty() => ClickEventHandler is null && DragStarted is null;

            private Point _downedPosition;
            private DateTime _downedTime;
            private bool _isClick;
        }
    }
```

当前这个类还有什么不足？没有提供外面可以设置点击的范围，也就是从按下开始可以移动的范围的值，以及运行点击的时间。从上面代码可以看到写的是 TimeSpan.MaxValue 也就是没有分开点击和长按的设置

另外方法里面还添加一个可选的委托是点击变拖动的事件，这个事件用来了解当前本来是点击的，但是点击的时候移动的距离判断为拖动

这个封装的方法没有用到什么框架里面的功能，因此换个框架，如 UWP 按照这个思路修改一点代码应该也可以做出来

现在这个逻辑放进了[HandyControl](https://github.com/HandyOrg/HandyControl/pull/414)欢迎小伙伴使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



# win10 uwp 获取按钮鼠标左键按下

我们可以使用`PointerPressed`获得鼠标右键按下，但是我们如何获得左键？

<!--more-->

其实UWP已经没有MouseLeftButtonDown，于是我们可以使用一个简单方法去获取鼠标左键按下。

我们在xaml 写一个 Button，给他名称MyButton

于是在xaml.cs写一个 PointerPressed 函数，把它在构造添加给我们的 Button 的 PointerPressedEvent

		

```C#
        public MainPage()
        {
            this.InitializeComponent();
            MyButton.AddHandler(PointerPressedEvent,
                new PointerEventHandler(Button_OnPointerPressed), true);
        }

        private void Button_OnPointerPressed(object sender, PointerRoutedEventArgs e)
        {
            
        }

```

如果这是大家在函数用断点，可以看到我们左键和右键按下都会触发。

但是我们如何知道我们按下的是左键还是右键？

我们可以使用e.Handle，如果是true，那么是左键，如果是false那么是右键。

这方法简单。

但是这不是一个好方法，我们可以使用GetCurrentPoint的IsLeftButtonPressed来知道我们是左键按下
		

```C#
            var temp = e.GetCurrentPoint(sender as Button);

            if (temp.Properties.IsLeftButtonPressed)
            {
                _leftMouse = true;
            }

```

如果我们只要左键，那么我们可以使用Tapped

Tap可以获得是鼠标、按下的点，这在弹出时有用。





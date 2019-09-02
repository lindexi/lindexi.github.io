
# win10 uwp 获取按钮鼠标左键按下

我们可以使用`PointerPressed`获得鼠标右键按下，但是我们如何获得左键？
UWP 左键按下经常获取不到，本文：如何获得鼠标按下

<!--more-->



<div id="toc"></div>

其实 UWP 已经没有 MouseLeftButtonDown ，于是我们可以使用一个简单方法去获取鼠标左键按下。

我们在 xaml 写一个 Button，给他名称 MyButton

于是在 xaml.cs 写一个 PointerPressed 函数，把它在构造添加给我们的 Button 的 PointerPressedEvent

		

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

上面是添加一个路由 PointerEventHandler ，无论有没 e.handle 在之前都会使用这个事件。

如果这是大家在函数 Button_OnPointerPressed 断点，可以看到我们左键和右键按下都会触发。

但是我们如何知道我们按下的是左键还是右键？

我们可以使用 e.Handle 查看是否是 true ，如果是 true ，那么是左键，如果是 false 那么是右键。

这方法简单。

但是这不是一个好方法，我们可以使用 GetCurrentPoint 的 IsLeftButtonPressed 来知道我们是左键按下
		

```C#
            var temp = e.GetCurrentPoint(sender as Button);

            if (temp.Properties.IsLeftButtonPressed)
            {
                _leftMouse = true;
            }

```

如果我们只要左键，那么我们可以使用Tapped

Tap可以获得是鼠标、按下的点，这在弹出时有用。

参见：[UWP开发大坑之---路由事件](https://kljzndx.github.io/My-Blog/UWP%E5%BC%80%E5%8F%91%E5%A4%A7%E5%9D%91%E4%B9%8B---%E8%B7%AF%E7%94%B1%E4%BA%8B%E4%BB%B6)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
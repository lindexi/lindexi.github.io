# win10 uwp 获得Slider拖动结束的值


本文讲的是如何获得Slider移动结束的值，也就是触发移动后的值。如果我们监听ValueChanged，在我们鼠标放开之前，只要拖动不放，那么就不停触发，而我们可能要得到事件是拖动结束后，那么我们如何获得。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

在WPF，我们可以使用`Thumb.DragCompleted`，连接：[http://stackoverflow.com/questions/723502/wpf-slider-with-an-event-that-triggers-after-a-user-drags](http://stackoverflow.com/questions/723502/wpf-slider-with-an-event-that-triggers-after-a-user-drags)，这个在UWP没有，所以我们没法使用这个。

但是可以使用鼠标放开的值，在 UWP 把触摸放开、鼠标这些叫 Pointer 那么是否监听 PointerReleased 就可以获得鼠标放开，实际监听也拿不到事件。

开始写一个简单页面

```csharp
      <Slider PointerReleased="UIElement_OnPointerReleased"></Slider>

```

然后后台使用 F12 生成代码，不需要写什么，但是在函数开始写断点，断点可以按 F9 就可以。这时候使用 F5 运行项目，可以看到，在鼠标松开不会进入断点。

那么是否有其他事件可以使用？实际上只有 SizeChanged ，他无法获得鼠标松开的值，也就是拖动结束的值。所以还是需要使用 PointerReleased ，为何这个函数不会进入？因为ms在注释写 请注意，并不保证 **Press** 操作结束会激发 Windows.UI.Xaml.UIElement.PointerReleased 事件；可能会改为激发其他事件。如果需要监听这个事件需要使用代码。

原因就是在底层进入 PointerReleased 使用了 e.Handle = true ，在 UWP 的路由事件，如果一个元素已经被设置事件处理，那么一般添加的事件函数就不会执行，如果需要这个事件函数执行，那么需要使用后台代码说无论是否在底层有处理，都需要触发，这样就可以使用在鼠标放开拿到值。

首先修改界面，给元素名称，然后打开后台代码，添加一段特殊的代码。

```csharp
      <Slider x:Name="Slider"></Slider>

        public MainPage()
        {
            this.InitializeComponent();
            Slider.AddHandler(UIElement.PointerReleasedEvent /*哪个事件*/, new PointerEventHandler(UIElement_OnPointerReleased) /*使用哪个函数处理*/, true /*如果在之前处理，是否还使用函数*/);
        }
```

这样，垃圾ms在底层处理，现在还是可以获得，因为设置了如果在之前处理，还使用定义的函数，这时在 UIElement_OnPointerReleased 就可以获得鼠标松开的值。

参见：[UWP开发大坑之---路由事件 - 快乐 就在你的心 的博客](https://kljzndx.github.io/My-Blog/2017/05/04/UWP%E5%BC%80%E5%8F%91%E5%A4%A7%E5%9D%91%E4%B9%8B-%E8%B7%AF%E7%94%B1%E4%BA%8B%E4%BB%B6/)

源代码：[[免费]SlideMove 1.0-CSDN下载](http://download.csdn.net/download/lindexi_gd/9979362)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
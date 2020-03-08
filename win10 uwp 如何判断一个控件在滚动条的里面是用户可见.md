# win10 uwp 如何判断一个控件在滚动条的里面是用户可见

在 UWP 中如何知道一个元素是在滚动条的显示大小内用户可以看到这个控件？如果需要在控件在滚动条里面用户可以看到的时候触发某个事件，在用户看不到的时候触发另一个事件可以怎么做？

<!--more-->
<!-- CreateTime:2019/11/29 8:42:36 -->

<!-- csdn -->

昨天[星期八再娶你](https://www.cnblogs.com/hupo376787) 大佬问我如何判断在滚动条内可以看到某个元素，他需要在滚动条里面放一个视频播放器，在用户看不到这个播放器的时候自动停下这个播放器

在 UWP 的判断会比在 WPF 中复杂一些，我写过[WPF 如何判断一个控件在滚动条的里面是用户可见](https://blog.lindexi.com/post/WPF-%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E4%B8%80%E4%B8%AA%E6%8E%A7%E4%BB%B6%E5%9C%A8%E6%BB%9A%E5%8A%A8%E6%9D%A1%E7%9A%84%E9%87%8C%E9%9D%A2%E6%98%AF%E7%94%A8%E6%88%B7%E5%8F%AF%E8%A7%81.html )但是在 UWP 中的小伙伴，也就是做 UWP 的大佬对 API 的设计会更加诡异

在 UWP 没有 ScrollChanged 事件只有[ScrollViewer.ViewChanged](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.scrollviewer.viewchanged?wt.mc_id=MVP) 事件，但是这个事件和 WPF 的触发不相同的在于，如果我有外层的控件修改了滚动条的大小，不会触发这个事件。在 [ScrollViewer.ViewChanged](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.scrollviewer.viewchanged?wt.mc_id=MVP) 只有在用户滚动或缩放完成之后才会触发，同时这个事件的参数[ScrollViewerViewChangedEventArgs](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.scrollviewerviewchangedeventargs?wt.mc_id=MVP ) 只有一个表示现在是用户交互的过程还是结束的变量，所以通过这个事件判断控件是否在滚动条可见是不可靠的

昨天[星期八再娶你](https://www.cnblogs.com/hupo376787) 大佬告诉我一个可以使用的方法是通过[LayoutUpdated](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.frameworkelement.layoutupdated?wt.mc_id=MVP ) 事件拿到触发，在布局属性修改的时候、在窗口修改的时候在运行时的布局的时候都会触发这个事件

在[LayoutUpdated](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.frameworkelement.layoutupdated?wt.mc_id=MVP )可以在控件第一次加载的时候触发，可以在用户滚动的时候触发

在 [LayoutUpdated](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.frameworkelement.layoutupdated?wt.mc_id=MVP ) 通过判断控件的左上角坐标和控件的大小可以判断用户是否可以看到这个控件

在 UWP 拿到一个控件相对于上一层控件的左上角坐标可以使用下面方法

```csharp
            var top = control.TransformToVisual(StackPanel).TransformPoint(new Point());
```

这个方法和 WPF 的 TranslatePoint 方法相同

判断滚动条可见大小不能从方法的参数拿到，需要直接拿滚动条控件，这样会存在一个坑在于时机的问题，和 WPF 不相同，此时的事件很难做到精确拿到滚动条的当前的大小和移动距离

```csharp
    var viewBounds = new Rect(new Point(ScrollViewer.HorizontalOffset, ScrollViewer.VerticalOffset), new Size(ScrollViewer.ViewportWidth, ScrollViewer.ViewportHeight));
```

在 UWP 的矩形判断里面的方法已经不存在，需要自己写一个辅助方法

```csharp
        private static bool RectIntersects(Rect a, Rect b)
        {
            return !(b.Left > a.Right
                || b.Right < a.Left
                || b.Top > a.Bottom
                || b.Bottom < a.Top);
        }
```

判断两个矩形是否相交就可以知道控件是否用户可以看到，如果想判断用户可以完全看到这个控件需要再写一个 Contain 方法

```csharp
        private void CheckControlShow()
        {
            UIElement control = TextBlock;

            var top = control.TransformToVisual(StackPanel).TransformPoint(new Point());
            var controlBounds = new Rect(top, control.DesiredSize);

            var viewBounds = new Rect(new Point(ScrollViewer.HorizontalOffset, ScrollViewer.VerticalOffset), new Size(ScrollViewer.ViewportWidth, ScrollViewer.ViewportHeight));

            if (RectIntersects(viewBounds, controlBounds))
            {
                Debug.WriteLine("歪楼");
            }
            else
            {
                Debug.WriteLine("不歪楼");
            }
        }
```

如果觉得 LayoutUpdated 触发的次数实在太多，那么请使用[FrameworkElement.SizeChanged](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.frameworkelement.sizechanged?wt.mc_id=MVP ) 和 [ScrollViewer.ViewChanged](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.scrollviewer.viewchanged?wt.mc_id=MVP) 事件同时使用，这样就可以拿到用户滚动和修改大小

[uwp - XAML ScrollViewer's child bring into view event - Stack Overflow](https://stackoverflow.com/q/55862430/6116637 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

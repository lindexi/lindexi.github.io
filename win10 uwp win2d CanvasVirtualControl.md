# win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl

本文来告诉大家 CanvasVirtualControl ，在什么时候使用这个控件。

<!-- csdn -->

<!-- <div id="toc"></div> -->

<!--more-->
<!-- CreateTime:2019/1/4 14:15:09 -->


<!-- 标签：uwp,win2d,渲染 -->

在之前的入门教程[win10 uwp win2d 入门 看这一篇就够了](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html )我直接用的是`CanvasControl`，实际上可以使用的画布还有下面两个

 - [CanvasAnimatedControl](https://microsoft.github.io/Win2D/html/T_Microsoft_Graphics_Canvas_UI_Xaml_CanvasAnimatedControl.htm )

 - [CanvasVirtualControl](https://microsoft.github.io/Win2D/html/T_Microsoft_Graphics_Canvas_UI_Xaml_CanvasVirtualControl.htm )

虽然本文主要告诉大家`CanvasVirtualControl`但是也是会告诉大家什么时候用哪个

## CanvasAnimatedControl

如果使用 CanvasControl ，那么只会在一开始使用了 drawn ，如果需要重新更新就需要通过调用 `Invalidate` 。

如果有很多次调用 `Invalidate` 会自动合并为一次，所以不能把  `Invalidate` 调用数和 draw 触发数作为相等。

如果是为了做动画需要不停调用 `Invalidate` ，在 UWP 比较好的方法是使用 CanvasAnimatedControl 这个可以到每秒 60 帧，而且在用户设备比较差的时候会降低调用频率。

所以做动画的时候需要不停触发重新渲染就使用 CanvasAnimatedControl ，关于这个控件，请看[win10 uwp 萤火虫效果](https://lindexi.gitee.io/post/win10-uwp-%E8%90%A4%E7%81%AB%E8%99%AB%E6%95%88%E6%9E%9C.html )

[Win2D 中的游戏循环：CanvasAnimatedControl](https://blog.csdn.net/WPwalter/article/details/84585594 )

## CanvasVirtualControl

和 CanvasAnimatedControl 频繁重新画不相同的，在 CanvasVirtualControl 的使用范围是很少刷新

如果满足下面任何条件就建议使用 CanvasVirtualControl 而不是 CanvasControl 因为这时的性能比较好

 - 如果你准备画一个非常大的图片

 - 不希望使用很多时间去画看不见的部分

 - 不想把整个图片都放在内存

因为 CanvasVirtualControl 使用位图虚拟化，所以不需要在所有的时候都把位图放在内存中，只有在需要显示的地方才是有效的，存放在内存的，对于不显示的地方是不放在内存，不画出来的。

在一个原来不显示的地方变为显示时就会触发[RegionsInvalidated](https://microsoft.github.io/Win2D/html/E_Microsoft_Graphics_Canvas_UI_Xaml_CanvasVirtualControl_RegionsInvalidated.htm )事件，这时就可以画出这部分。

```csharp
void OnRegionsInvalidated(CanvasVirtualControl sender, CanvasRegionsInvalidatedEventArgs args)
{
    foreach (var region in args.InvalidatedRegions)
    {
        using (var ds = sender.CreateDrawingSession(region))
        {
            // draw the region
        }
    }
}
```

所以通过这个方法就可以不需要手动去判断哪些是显示的，只要触发了，就是可以画出的。那么怎么知道触发的显示的矩形？实际上从`args.InvalidatedRegions`就是拿到一个 Rect ，通过这个就可以判断需要显示的是哪个。

很多时候使用 CanvasVirtualControl 都是和 ScrollViewer 一起使用

```csharp
 <ScrollViewer>
  <canvas:CanvasVirtualControl Width="10000" Height="10000" RegionsInvalidated="OnRegionsInvalidated" />
</ScrollViewer>
``` 

所以在滚动的时候就可以判断哪些需要显示，通过只画显示的来提高性能。

当然在页面大小变化或者 CanvasVirtualControl 需要修改大小，还是需要调用 invalidated 来重新画

```csharp
void VirtualControl_SizeChanged(object sender, SizeChangedEventArgs e)
{
    VirtualControl.Invalidate();
}
```

## 其他博客

[win10 uwp win2d 入门 看这一篇就够了](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html )

win2d 毛玻璃：[win10 uwp 毛玻璃](https://lindexi.gitee.io/post/win10-uwp-%E6%AF%9B%E7%8E%BB%E7%92%83.html )

[win2d 画出好看的图形](https://lindexi.gitee.io/post/win2d-%E7%94%BB%E5%87%BA%E5%A5%BD%E7%9C%8B%E7%9A%84%E5%9B%BE%E5%BD%A2.html )

[win10 uwp 萤火虫效果](https://lindexi.gitee.io/post/win10-uwp-%E8%90%A4%E7%81%AB%E8%99%AB%E6%95%88%E6%9E%9C.html )

[win2d 图片水印](https://lindexi.gitee.io/post/win2d-%E5%9B%BE%E7%89%87%E6%B0%B4%E5%8D%B0.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

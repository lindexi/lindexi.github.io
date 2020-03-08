# wpf VisualBrush 已知问题

本文告诉大家，visualBrush 已知 bug ，希望大家使用 VisualBrush 时可以知道

<!--more-->
<!-- CreateTime:2019/7/12 21:07:41 -->

<!-- csdn -->

1. 如果把 VisualBrush 绑定的是在元素加入到视觉树前，那么在元素加入到视觉树之后移除视觉树，VisualBrush 就不会自动刷新

1. 如果把没有加入视觉树的元素加入到 VisualBrush 绑定，之后把元素加入视觉树，再移除，再加入，这时可能 VisualBrush 不再刷新

1. 如果在 VisualBrush 获取到元素之后，设置元素的 visibility 为 Collapsed 那么 VisualBrush 不会更新布局，详细请看[The VisualBrush only refresh the visual but not the layout when the Visual visibility changes](https://github.com/dotnet/wpf/issues/1241 )

1. 如果元素绑定 VisualBrush 然后对元素使用 RenderTargetBitmap 就会让 VisualBrush 无法使用。

解决方法，设置 VisualBrush 的 Visual 为空再设置元素

```csharp
var visual = visualBrush.Visual;
visualBrush.Visual = null;
visualBrush.Visual = visual;
```

参见：[https://stackoverflow.com/a/3073378/6116637](https://stackoverflow.com/a/3073378/6116637)

[https://stackoverflow.com/a/13182210/6116637](https://stackoverflow.com/a/13182210/6116637)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

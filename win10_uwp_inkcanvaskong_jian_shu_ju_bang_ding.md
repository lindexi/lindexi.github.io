# win10 uwp InkCanvas控件数据绑定

本文主要说如何绑定InkCanvas，让笔画变化的时候我们可以知道。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

我们本来的InkCanvas没有提供笔画绑定，所以我们自己写

```csharp
using Windows.UI.Input.Inking;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

public static class InkCanvasBinder
{
    public static InkStrokeContainer GetInkStrokes(DependencyObject obj) => 
        obj.GetValue(InkStrokesProperty) as InkStrokeContainer;

    public static void SetInkStrokes(DependencyObject obj, InkStrokeContainer value) => 
        obj.SetValue(InkStrokesProperty, value);

    public static DependencyProperty InkStrokesProperty = DependencyProperty.RegisterAttached(
        "InkStrokes", typeof(InkStrokeContainer), typeof(InkCanvasBinder),
        new PropertyMetadata(null, InkStrokesProperty_PropertyChanged));

    private static void InkStrokesProperty_PropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        var inkCanvas = d as InkCanvas;
        if (inkCanvas != null) inkCanvas.InkPresenter.StrokeContainer = e.NewValue as InkStrokeContainer;
    }
}
```

我们使用InkCanvas

```xml
<InkCanvas local:InkCanvasBinder.InkStrokes="{x:Bind AnInkStrokeContainer}" />
```

参见：https://github.com/Microsoft/Windows-task-snippets/blob/master/tasks/InkCanvas-data-binding.md

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


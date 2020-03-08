# WPF 可获得焦点属性

本文来告诉大家 WPF 的可获得焦点属性，如果希望一个元素可以获得键盘输入，那么就需要一个元素是可以获得焦点，而且焦点就在元素上。


<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

WPF 的可获得焦点元素是 Focusable，这个属性是所有继承 `IInputElement` 的类都有，也就是所有的 UIElement 都可以设置 Focusable ，如果一个元素设置了 `Focusable = false` 那么这个元素就不能使用 Tab 把焦点放在这个元素。不能获得键盘的输入，但是可以获得鼠标输入。

如果需要获得键盘输入，就需要焦点在这个元素，很多时候在鼠标点击的元素就自动设置焦点是这个元素，但是如果这个元素`Focusable = false`就不会设置焦点。

## 默认的属性

如果反编译 WPF 可以看到默认的值是 false ，但不是所有的控件都是 false ，下面我来告诉大家哪些控件默认是 false ，哪些是 true，方便大家快速去查元素

![](http://image.acmx.xyz/lindexi%2F20185151537382265.jpg)

默认是 true 的类

 - Button
 - Calendar
 - ComboBox
 - DataGrid
 - DatePicker
 - ListBox
 - RichTextBox
 - Slider
 - TabControl
 - TextBox
 - TreeView
 - Window


默认是 false 的控件

 - Canvas
 - DockPanel
 - Grid
 - Image
 - Label
 - ProgressBar
 - ScrollBar
 - Separator
 - Shape 类型 (Ellipse, Line, Path, Polygon, Polyline, Rectangle)
 - StackPanel
 - TextBlock
 - UniformGrid
 - Viewport3D
 - WrapPanel

如果觉得上面的类很多，无法记住，那么简单的方法是记下默认是 True 的类：具有交互的控件；默认是 False 的类：没有交互，仅供显示或布局的元素。

参见：[#623 – Focusable Property Indicates Whether a Control Can Receive Focus](https://wpf.2000things.com/2012/08/13/623-focusable-property-indicates-whether-a-control-can-receive-focus/ )


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

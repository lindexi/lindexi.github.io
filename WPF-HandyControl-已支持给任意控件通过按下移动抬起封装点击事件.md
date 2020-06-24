
# WPF HandyControl 已支持给任意控件通过按下移动抬起封装点击事件

著名的 HandyControl 已经支持给任意控件通过按下移动抬起事件，封装点击事件

<!--more-->


<!-- 发布 -->

在 [HandyControl](https://github.com/HandyOrg/HandyControl/pull/414) 的这个 [PR](https://github.com/HandyOrg/HandyControl/pull/414) 添加了 InputClickHelper 类，这个类提供了使用控件的按下移动抬起事件封装为点击事件

使用方法：

```csharp
Install-Package HandyControl
```

给任意控件 element 附加按下移动抬起封装点击事件，下面代码的 uiElement 是一个 UIElement 控件

```csharp
HandyControl.Tools.InputClickHelper.AttachMouseDownMoveUpToClick(uiElement, UIElement_OnClicked);

        private void UIElement_OnClicked(object sender, EventArgs e)
        {
            
        }
```

此外，在 AttachMouseDownMoveUpToClick 方法还提供了按下过程中，用户移动鼠标或触摸触发的点击事件打断作为拖拽事件。使用方法如下

```csharp
HandyControl.Tools.InputClickHelper.AttachMouseDownMoveUpToClick(uiElement, UIElement_OnClicked, UIElement_OnDragStarted);

        private void UIElement_OnDragStarted(object sender, EventArgs e)
        {
            
        }
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
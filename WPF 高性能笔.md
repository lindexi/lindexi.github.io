# WPF 高性能笔

本文告诉大家WPF的INK的实现，和如何做一个高性能的笔。

<!--more-->
<!-- csdn -->

本文主要告诉大家如何继承 StylusPlugIn 来做高性能的笔。

先创建一个类 TtkSwvlypxm 继承 StylusPlugIn ，那么可以通过重写获得

 - OnAdded 被添加时

 - OnRemoved 

 - OnStylusEnter 触摸时

 - OnStylusLeave

 - OnStylusDown

 - OnStylusMove

 - OnStylusUp

 - OnStylusDownProcessed 可以判断是否失焦

 - OnStylusUpProcessed

那么在这里类，几乎可以不写代码就获得触摸事件，从这里获得触摸事件比路由会快，因为这里是 rawStylusInput ，没有处理的事件，可以获得触摸宽度和触摸的元素。

那么如何加入这个类？

使用 InkPresenter 创建一个类，这个类用来显示笔迹，之后需要在添加 InkPresenter 的类 上添加事件

例如 SlwqntthSpeswbrj 添加了 InkPresenter ，那么需要使用下面的代码

```csharp
            var dynamicRenderer = new TtkSwvlypxm();

            dynamicRenderer.Enabled = true;

            SlwqntthSpeswbrj.StylusPlugIns.Add(dynamicRenderer);
```

这样尝试在触摸时就可以获得触摸事件，因为获得事件比较快，所以性能比较高。

其他的代码因为在公司使用，所以我就不写下来

只要获得了触摸事件，要画出来是很简单。

如果支持多指，其实只需要多创建 TtkSwvlypxm 就可以支持多指

可能存在的问题，刚才有附加的代码 StylusPlugIns.Add ，实际上 StylusPlugIns 是 UIElement 的保护，所以需要写一个函数把这个属性给外面。

如果需要移除，那么请设置                    dynamicRenderer.Enabled = false; 直接移除会出现直接退出

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
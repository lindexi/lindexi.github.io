# WPF 高性能笔

本文告诉大家WPF的INK的实现，和如何做一个高性能的笔。

<!--more-->
<!-- CreateTime:2019/11/29 10:20:51 -->

<!-- csdn -->
<!-- 标签：笔迹，WPF -->

高性能的笔迹在 WPF 包含两个部分，一个是就是输入，第二个就是渲染。

如果需要经过路由事件才收到输入，如果有人在路由事件做了很多需要很长事件的代码，那么等待用户的路由事件就会使用很长的时间。

如果需要等待主界面的布局也就是如果主线程卡住了，就需要等待主线程才可以渲染。

所以按照原来的元素的输入渲染是无法做到高性能的，那么 WPF 的笔迹是如何做到很快？这里需要用到两个科技，一个就是输入使用 StylusPlugin 一个就是使用另一个 UI 线程解决渲染的速度。

这里说的另一个 UI 线程解决渲染速度而不是使用另一个渲染线程是因为在 WPF 是分开主线程渲染线程，具体请看 WPF 渲染原理。

为什么 Stylusplugin  可以做到高性能？

这个需要从触摸开始讲。在我的另一篇博客有告诉大家从触摸到事件，在 WPF 是通过触摸线程拿到触摸信息。

在触摸线程获取触摸消息的时候，会根据收到的触摸消息转发不同的方法。

在转发的过程，在 WPF 会通过 StylusPlugins 里静态字典，存放用户设置的类。在触摸线程会通过判断触摸点时候在命中对应的元素矩形区判断当前时候命中到这个元素。这里判断命中测试和 WPF 说的命中测试使用的不是同相同的方法，这里只是简单获取每个界面元素的矩形，然后用触摸的点坐标判断是否在这个矩形内，也就是不判断元素是否被其他的元素挡住。所以这个判断方法不需要遍历视觉树，性能相对很高。

这是就为什么使用 StylusPlugin 的获取输入性能比较快。因为这个过程是从触摸线程拿到的，而且触摸线程在执行 StylusPlugin 后才执行到路由事件的代码，使用 StylusPlugin 的速度会比路由事件快很多，加上路由事件需要做命中测试，可能用户会在路由事件做很多事件。而 Stylusplugin 只是从触摸线程拿到，完全不需要等用户在路由事件代码。

下面就是在触摸线程调用 Stylusplugin 的代码

在使用渲染这里用另一个线程做 UI 线程，在 WPF 不是只有主线程可以做 UI 线程，这里的 UI 线程和渲染线程是不相同，因为渲染线程是收集 UI 线程发过来的数据然后才进行渲染。

这里通过 VisualHost 的方法创建一个 UI 线程，在这个线程计算笔迹，然后添加到这个线程的元素，通过这个方式可以在主线程做其他代码的时候还可以快速在用户触摸的时候告诉渲染线程。

在 WPF 的 笔迹是没有额外创建一个线程作为另一个 UI 线程，而是直接将触摸收集线程作为另一个 UI 线程。当然这个方法如果没用好可能就会在用户多个手指书写时无法做到足够高的速度。

如果要做高性能的笔必须要了解 WPF 的触摸和渲染原理，具体请看[WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html ) 和 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

于是下面告诉大家如何做出一个高性能的笔。

本文主要告诉大家如何继承 StylusPlugIn 来做高性能的笔。需要知道 StylusPlugIn 提供了底层的触摸事件，这个事件从 Wisp 进程获得数据然后直接给框架，然后给 UIElement 所以继承StylusPlugIn可以拿到比路由事件更快。

为什么说 StylusPlugIn 拿到比 路由事件更快，这需要了解一下 lnk 的底层。

如果直接从 StylusDown 事件拿到，那么这个事件是经过 WispLogic 和 StylusLogic 处理之后的值才会传给 Stylus.StylusDownEvent ，然后使用路由事件的方式，先经过隧道然后冒泡才到 UIElement ，如果有人在到 UIElement 之前写了代码，或者主线程做了其他不清真的（while xx）那么用户触摸到 UIElement 收到消息就过去很久。

那么StylusPlugIn为什么会比较快，原因是 StylusPlugIn 没有经过那么多处理，也没有经过隧道，而且他可能还不在主线程，不管主线程被写了多少代码，他这个线程都不会被影响。调用的线程级别是输入，除非主线程真的占用整个CPU，不然主线程的代码对这个线程影响很小。

因为 StylusPlugIn 是从 StylusInput 修改来的，所有的 UIElement 都有 StylusPlugIns  属性，但是这个属性是只有继承 UIElement 的类才可以拿到。从 StylusPlugIn 拿到的数据就是系统拿到的 xy 点和触摸压力，还有触摸宽度。但是这里的宽度是需要反射才可以拿到，不是所有的触摸屏都可以报告触摸宽度。

如果需要加入到 StylusPlugIn 首先需要继承 StylusPlugIn

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

如果需要移除，那么请设置`dynamicRenderer.Enabled = false;` 直接移除会出现直接退出

那么使用 StylusPlugIn 的作用除了做高性能的笔之外还有什么作用？实际上可以看到这个方法可以用来过滤输入，因为他在路由事件之前，而且可以修改点，所以用它来修改过滤。

自己定义的 StylusPlugIn 实际上作为笔迹还是存在很多坑，所以一般都是继承 DynamicRenderer ，这个类对输入做了很多处理，当然也存在一些坑。

参见：[Intercepting Input from the Stylus](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/intercepting-input-from-the-stylus )

[WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html ) 

[WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

其他自己写的笔迹算法

- [原笔迹手写实现平滑和笔锋效果之:笔迹的平滑(一) - 88911562 - 博客园](http://www.cnblogs.com/zl03jsj/p/8047259.html )

- [原笔迹手写实现平滑和笔锋效果之:笔迹的平滑(二) - 88911562 - 博客园](https://www.cnblogs.com/zl03jsj/p/8048102.html )

- [原笔迹手写实现平滑和笔锋效果之:笔锋效果(三)[完结篇] - 88911562 - 博客园](https://www.cnblogs.com/zl03jsj/p/8051912.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 